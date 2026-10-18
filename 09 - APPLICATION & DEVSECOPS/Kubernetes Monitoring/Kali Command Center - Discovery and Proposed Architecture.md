# Kali Command Center: Discovery and Proposed Architecture

**Phase:** 1–3 complete (read-only discovery, architecture proposal, risks)  
**Snapshot:** 2026-10-17, America/New_York  
**Host:** Kali GNU/Linux Rolling 2026.3, kernel 7.1.5, x86_64, 8 logical CPUs, America/New_York

> This report contains no credentials, environment-variable values, private keys, or log message contents. Discovery commands were read-only. No packages were installed and no application, service, firewall, network, or monitoring configuration was changed.

## Executive summary

This workstation already has a substantial monitoring stack: Prometheus, Grafana, Alertmanager, kube-state-metrics, Node Exporter, and Trivy Operator run in the existing `kind-devsecops` cluster. Prometheus and Grafana respond locally, and all 20 active Prometheus scrape targets reported healthy. Host Netdata 2.11.0 nightly also runs and exposes Prometheus-format host metrics. A user service named `docker-container-exporter` exists, but its collection path and Prometheus integration have not yet been confirmed.

The existing Kubernetes dashboards mostly describe the **kind node and its pods**, not the Kali host and all Docker workloads. The host already has useful data available from Netdata, so adding a second Grafana/Prometheus stack or a new Kubernetes cluster would duplicate services and consume storage and CPU. The proposed approach is to extend the current stack after approval, use an explicit project/service registry, and avoid Docker socket access.

Two storage conditions need attention before increasing collection: `/mnt/vmstorage` is 94% full (7.7 GiB free) and `/` is 83% full (38 GiB free). Prometheus currently retains 10 days with no size cap and stores about 549 MB of blocks plus 104 MB of WAL. Prometheus and Grafana currently use ephemeral `emptyDir` volumes; no PVCs were returned by cluster discovery. A pod restart can therefore lose Prometheus history and Grafana database state.

## System baseline

| Area | Observation |
| --- | --- |
| CPU and memory | Intel Xeon E5-1630 v3, 8 logical CPUs; 46 GiB RAM, 28 GiB available; 976 MiB swap unused. A one-second CPU sample was 34.7%; 5-minute load average was 7.71 and CPU pressure `avg300` was 5.92%. These are snapshots, not long-term baselines. |
| Temperature | CPU package 58°C and GPU 53°C at inspection; no thermal alarm was reported. |
| Root filesystem | ext4, 233 GiB total, 183 GiB used, 38 GiB available (83%). Inodes 18% used. |
| VM storage | `/mnt/vmstorage`, ext4, 116 GiB total, 108 GiB used, 7.7 GiB available (94%). |
| Large data disk | `/mnt/elements`, exFAT, 4.6 TiB total, 1.2 TiB used, 3.4 TiB available (27%). Keep database/TSDB placement on a suitable POSIX filesystem; do not assume exFAT is suitable for Prometheus data. |
| Network | Active uplink `eth1` at `172.20.10.8`; `eth0` and `wlan0` were down. Several Docker bridges and a lab bridge are active. No routes or firewall rules were changed. |
| Docker | Docker 28.5.2 and Compose v5.3.0. 27 containers are accounted for; Docker reports 20.92 GB of images, 14.56 GB of volumes, 193 MB of container writable layers, and 716 MB of build cache. No cleanup was performed. |
| Current-boot errors | A snapshot found about 730 error-severity journal entries, mostly attributed to `kernel` and `bluetooth.service`. Message contents were not collected; this count needs contextual triage and should not be treated as a count of unique incidents. |
| Failed systemd units | `bluetooth-fix.service` and `systemd-networkd-wait-online.service`. The host uses NetworkManager; determine whether the wait-online failure is relevant before alerting on it. |
| Virtual machines | `vmrun list` reported zero running VMs. VMware tooling is present; GOAD files and VM storage exist. |

## Existing observability and persistence

| Component | Observed state and limitation |
| --- | --- |
| Grafana | Grafana API returned HTTP 200 at `127.0.0.1:13000`; Grafana runs in the kind cluster. The host port is a `kubectl port-forward`, not a native Grafana service. |
| Prometheus | `127.0.0.1:19090/-/healthy` returned HTTP 200; 20/20 active targets were `up`. Current head series: about 130,701. Retention: 10 days; size limit: unset (`0B`). |
| Prometheus data | Data path is `/prometheus`, backed by `emptyDir`; reported blocks were about 549 MB and WAL about 104 MB. This is ephemeral across pod recreation and has no configured size ceiling. |
| Grafana data | Grafana pod storage is also `emptyDir`. Dashboards are sidecar-provisioned from ConfigMaps, but other Grafana database state may not survive pod recreation. |
| Kind lifecycle | The existing `devsecops-control-plane` Docker container has restart policy `on-failure`; verify reboot behavior before relying on it for startup persistence. Existing UI/API port forwards are process-bound and are not persistent services. |
| Node Exporter | A Node Exporter target exists at the kind node address. It measures the kind node/container environment, not the complete Kali host. |
| Netdata | Host Netdata 2.11.0 nightly responds on port 19999 and its Prometheus exposition endpoint returns metrics. Host charts include CPU, RAM, disk space/inodes, network interfaces, and I/O pressure. Its listener currently binds all host addresses (`0.0.0.0:19999`), so restrict its reachability before using it as an internal scrape source. |
| Docker exporter | A user-level `docker-container-exporter.service` is active. Its output path and whether Prometheus consumes it remain unverified. Do not assume it provides complete container metrics yet. |
| Alerting | Alertmanager is part of the cluster stack. No external notification credentials or destinations were inspected or configured. |
| Logs | No Loki installation was confirmed. Start without centralized logs; add a log backend only if a bounded, redacted collection plan is justified. |

## Workload and project inventory

A directory is not evidence that an application is running. Projects without an identified process/container/endpoint should begin as **discovered / unconfigured / unknown**, not healthy or failed. Project files such as `.env` were not read.

| Project or service | Evidence and current state | Initial monitoring treatment |
| --- | --- | --- |
| `/var/www/CTFd` | Python application source, Dockerfile, Compose file and virtual environment; no matching active container was observed. Git worktree has local changes. | Discovered; runtime and endpoint unverified. |
| `/var/www/HackAfrica` | Compose-based Python project. `hackafrica-api-1` is restarting; worker is up; Redis and Mailpit report healthy. | Monitor dependencies separately; API is currently unhealthy/restarting, not green. Verify its health endpoint and failure reason during implementation. |
| `/var/www/cdc` | PHP/Composer and Node project files; no matching active container was observed. Git worktree has local changes. | Discovered; runtime and endpoint unverified. |
| `/var/www/html` | Static web content; no matching active host web service was confirmed. | Discovered; do not infer that it is served. |
| `/var/www/muhammed-camara` | Static site repository; no matching running service confirmed. | Discovered; unconfigured. |
| `/var/www/noftdigital` | Python/Django project files and Dockerfile; no matching active container observed. Git is behind its remote and has untracked scanner output. | Discovered; runtime and endpoint unverified. |
| `/var/www/ocd-mindmaps` | Static/site source repository; no matching running service confirmed. | Discovered; unconfigured. |
| `/var/www/wwcg` | PHP/Composer and Node project files; no matching active container observed. Git worktree has local changes. | Discovered; runtime and endpoint unverified. |
| AEGIS (`/home/camara/Aegis`, `/home/camara/src/aegis`) | Source directories exist. No matching container or system service was established by this snapshot. | Local code exists; runtime unknown. |
| Floci | Five local containers observed. Main, GCP, and Azure emulator containers report healthy; UI/proxy are up. Published ports include 4500, 4566–4567, 4577 and 4588. | Monitor confirmed local endpoints and container health only. |
| Mythic | Local container stack observed; core server, database, queue and UI components are up, many report healthy. UI/API ports include 7443 and several loopback-only ports. | Monitor availability/resource health only; do not collect C2 payloads, credentials, intercepted traffic, or task content. |
| GOAD / VMware | GOAD directory and VM storage exist; zero running VMs reported. | VM state is stopped at this snapshot; do not probe remote lab systems. |
| Wazuh | A Wazuh agent package file exists under a project directory, but no running manager/agent was confirmed. | Not registered as an active service without further evidence. |
| Portainer | Container is up; HTTPS port 9443 is published on all host interfaces. | Monitor reachability; leave firewall and publication unchanged during discovery. |
| Kubernetes Dashboard | Dashboard pods run in kind; local forwarding is active on port 8443. | Monitor only via its existing authenticated route. |
| Netdata | Host service is running and responds on port 19999, bound to all addresses. | Potential existing host-metric source; tighten access before Prometheus integration. |

## Port and storage risks

- Local Grafana `13000`, Prometheus `19090`, Kubernetes Dashboard `8443`, and the kind API `42625` are currently loopback-forwarded. Their availability depends on active `kubectl` forwarding processes.
- Port `3000` is already used on loopback by a Mythic web component. A second Grafana container using its default host port would conflict. Reuse the existing Grafana instance.
- Netdata `19999` is bound to all interfaces. Docker publishes services on all interfaces, including Portainer `9443`, Floci `4500/4566/4567/4577/4588`, Mythic `7443`, HackAfrica Redis `6381`, and Mailpit `1025/8025`. SSH `22` and an unidentified listener on `3306` are also bound to all interfaces. Binding is not the same as internet reachability; firewall policy was not inspected or changed.
- A dedicated local monitoring database on `/mnt/vmstorage` would compete for the remaining 7.7 GiB. `/` has 38 GiB free but is already 83% full. `/mnt/elements` is exFAT and is not the default recommendation for TSDB files.
- Existing Prometheus and Grafana have ephemeral pod storage; Prometheus has no storage-size limit. Persistent storage and a bounded retention size are needed before relying on long-term history.
- The `kind` node currently uses about 4.9 GiB RAM and had a 38% CPU snapshot in Docker stats. Avoid adding Loki, cAdvisor, duplicate host collectors, or another full Prometheus/Grafana stack during the first phase.

## Proposed architecture (pending approval)

1. **Extend the existing monitoring stack.** Keep the current kind cluster and Grafana/Prometheus/Alertmanager; do not install another cluster or a duplicate Grafana/Prometheus. Back up the current Helm values, ConfigMaps, dashboards, and Prometheus rules before any changes.
2. **Add Kali host metrics from the existing Netdata agent.** Validate that the kind Prometheus can reach the host bridge endpoint. Restrict Netdata’s listener to the kind Docker bridge address (or an equally narrow private path), then configure Prometheus to scrape only the required host metrics endpoint. Do not publish a new metrics port to the LAN. If this path cannot be secured and reached reliably, stop and compare a least-privilege host exporter design before installing anything.
3. **Bound and persist time-series data.** Preserve current 10-day retention initially and add a 2 GiB TSDB size cap, subject to review of actual growth. Add persistent claims for Prometheus and Grafana using the existing local-path storage, sized conservatively and backed by the Docker/kind storage on ext4. This will survive pod recreation, but not necessarily a deliberate kind-cluster deletion; full host-level durability needs a separate design. Do not place TSDB data on exFAT.
4. **Create a project/service registry.** Register the eight `/var/www` paths and confirmed local lab components. New paths start as `discovered/unconfigured`. Add only explicit HTTP/TCP checks, systemd units, or container names; never execute project code or scan arbitrary networks. Record last success, failure reason, response time and confidence.
5. **Use existing Docker observability first.** Audit the installed user-level exporter and Netdata’s container metrics before adding a collector. Do not mount the Docker socket into Grafana, Prometheus, or an unrelated container. Add cAdvisor only if a safe, bounded configuration is demonstrated to be necessary.
6. **Build dashboards from available metrics.** Create host executive, system performance, services/ports, Docker, projects, confirmed lab infrastructure, storage/capacity, network, and alert/reliability dashboards. Use explicit `unknown`, `stopped`, `unconfigured`, `healthy`, and `failing` states. Keep log panels out of the first rollout.
7. **Add local-only alerts.** Use the existing Alertmanager without external receivers. Start with exporter/host availability, sustained CPU or memory pressure, filesystem space/inodes, explicitly registered service checks, restarting containers, and Prometheus storage. Disk thresholds will immediately surface the current 94%-full VM filesystem; review severity before enabling notifications.
8. **Make access survive reboot.** After confirming Docker/kind startup semantics, add user-level systemd units for the loopback-only `kubectl port-forward` endpoints and ensure the kind node is started reliably. This is a persistent user-service change and is not applied yet.
9. **Defer centralized logs.** Do not deploy Loki initially. First validate metrics, dashboards, project checks and retention. If logs are added later, use an explicit allowlist, redaction, size/retention budgets and no collection of security-tool payloads or credentials.

## Approval gate and next stage

The discovery and proposed architecture are complete. **No system-wide changes have been applied.** The next stage changes persistent Kubernetes monitoring configuration, Netdata listener configuration, and user-level startup units. The request specification explicitly requires approval before persistent service changes, installations, firewall changes, or potentially disruptive work.

To proceed, approve this scope: extend the existing kind Prometheus/Grafana stack; restrict and scrape Netdata through the kind bridge; add bounded persistent Prometheus/Grafana storage; create a read-only project registry, explicit local health probes, dashboards and local alert rules; and make the loopback port forwards start through user-level systemd. No package installations, firewall edits, remote lab changes, project starts, Docker cleanup, or application workload changes are included in that scope.
