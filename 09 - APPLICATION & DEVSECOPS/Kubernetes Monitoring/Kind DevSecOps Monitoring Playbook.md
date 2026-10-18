---
title: Kind DevSecOps Monitoring Playbook
category: Operations Playbook
tags:
  - Kubernetes
  - kind
  - DevSecOps
  - Grafana
  - Prometheus
  - Trivy
  - Falco
date_created: 2026-10-17
status: ACTIVE
environment: kind-devsecops
---

# Kind DevSecOps Monitoring Playbook

Operational reference for the local `kind-devsecops` Kubernetes cluster. Commands below were matched to the observed cluster resources on 2026-10-17. Run `kubectl config current-context` first; do not assume the selected context is this cluster.

> [!warning] Protect the cluster
> The Docker container `devsecops-control-plane` is the Kubernetes node. Do not stop, remove, or recreate it to troubleshoot an individual app. That can stop or destroy the whole local cluster. Prefer Kubernetes workload commands below.

## What runs here

| Component | Namespace | Workload / service | Purpose |
|---|---|---|---|
| Grafana | `default` | `monitoring-grafana`, service `monitoring-grafana:80` | Dashboards at `http://localhost:13000` |
| Prometheus | `default` | StatefulSet `prometheus-monitoring-kube-prometheus-prometheus`, service `monitoring-kube-prometheus-prometheus:9090` | Metrics and scrape targets at `http://localhost:19090` |
| Alertmanager | `default` | StatefulSet `alertmanager-monitoring-kube-prometheus-alertmanager`, service `monitoring-kube-prometheus-alertmanager:9093` | Alert routing at `http://localhost:19093` |
| Trivy Operator | `trivy-system` | Deployment `trivy-operator` | Kubernetes and image security reports; metrics service `trivy-operator:80` |
| Falco | `default` | DaemonSet `falco`, service `falco-metrics:8765` | Runtime threat detection and metrics |
| Kubernetes Dashboard | `kubernetes-dashboard` | Deployment and service `kubernetes-dashboard:443` | Kubernetes web UI at `https://localhost:8443` |
| Gatekeeper | `gatekeeper-system` | `gatekeeper-audit`, `gatekeeper-controller-manager` | Admission policy and compliance auditing |
| nginx sample | `default` | Deployment/service `nginx` | Sample workload; inspect before using or removing |

The observed services `monitoring-grafana`, Prometheus, Alertmanager, and Dashboard are `ClusterIP`, so they are not directly exposed on the host. A port-forward provides temporary local access.

## 1. Verify context and cluster health

```bash
# Show the cluster kubectl will control. Expected: kind-devsecops.
kubectl config current-context

# Show cluster API endpoints and node readiness.
kubectl cluster-info
kubectl get nodes -o wide

# List every pod, including namespace and restart count.
kubectl get pods -A -o wide

# List all services and their in-cluster ports.
kubectl get services -A

# Show deployments, StatefulSets and DaemonSets with desired/ready counts.
kubectl get deploy,sts,ds -A

# Find pods that are not Running/Completed, or workloads that are not ready.
kubectl get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded
kubectl get events -A --sort-by=.lastTimestamp | tail -50
```

**Why:** This establishes the active context and gives a quick health snapshot before making changes. A high `RESTARTS` count records past restarts; it does not by itself mean the container is failing now. Check current `READY`/`STATUS`, restart timestamps, events, and previous logs.

## 2. Open the monitoring UIs

Run each port-forward in its own terminal and leave that terminal open. Stop a forward with `Ctrl+C`.

```bash
# Grafana dashboards
kubectl -n default port-forward svc/monitoring-grafana 13000:80 --address 127.0.0.1

# Prometheus expression browser and targets
kubectl -n default port-forward svc/monitoring-kube-prometheus-prometheus 19090:9090 --address 127.0.0.1

# Alertmanager UI
kubectl -n default port-forward svc/monitoring-kube-prometheus-alertmanager 19093:9093 --address 127.0.0.1

# Kubernetes Dashboard HTTPS UI
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard 8443:443 --address 127.0.0.1
```

Open these URLs on the **same host where the port-forward command is running**:

- Grafana: [http://localhost:13000](http://localhost:13000)
- Prometheus: [http://localhost:19090](http://localhost:19090)
- Alertmanager: [http://localhost:19093](http://localhost:19093)
- Kubernetes Dashboard: [https://localhost:8443](https://localhost:8443) (the browser may warn about its local certificate)
- Trivy dashboard: [http://localhost:13000/d/trivy-operator-overview/trivy-operator-overview?orgId=1&from=now-24h&to=now&timezone=browser&refresh=30s](http://localhost:13000/d/trivy-operator-overview/trivy-operator-overview?orgId=1&from=now-24h&to=now&timezone=browser&refresh=30s)
- Falco dashboard: open Grafana, then use Dashboards search for “Falco”.

**Why:** A `ClusterIP` is reachable only inside Kubernetes. Port-forward bridges one local host port to that service; it is temporary and ends if the command or terminal exits. `--address 127.0.0.1` keeps the forwarded port local to this machine.

## 3. Dashboard and authentication notes

Grafana uses the admin credentials stored in Kubernetes Secret `monitoring-grafana` in namespace `default`. Do not paste credentials into notes, shell history, tickets, or chat. To retrieve them locally when needed:

```bash
# Print only the username.
kubectl get secret monitoring-grafana -n default -o jsonpath='{.data.admin-user}' | base64 -d; echo

# Reveal the password only in your private terminal; avoid screenshots/logging.
kubectl get secret monitoring-grafana -n default -o jsonpath='{.data.admin-password}' | base64 -d; echo
```

The Dashboard `admin-user` ServiceAccount is bound to `cluster-admin` in this lab. Generate an expiring token when you need to log in:

```bash
# Create a one-hour token; paste the output directly into the Dashboard Token form.
kubectl create token admin-user -n kubernetes-dashboard --duration=1h

# Confirm the account has cluster-wide pod read permission without printing a token.
kubectl auth can-i get pods --all-namespaces --as=system:serviceaccount:kubernetes-dashboard:admin-user
```

**Why:** Modern Kubernetes issues short-lived ServiceAccount tokens. Old copied tokens expire or can be invalid for this cluster. Do not save the generated bearer token in this playbook. Cluster-admin is broad access; use it only for this local lab and consider replacing it with a least-privilege account if exposing the Dashboard beyond localhost.

## 4. Inspect a workload

```bash
# List pods in the chosen namespace.
kubectl get pods -n default -o wide

# Describe a pod: events, probes, mounts, image and scheduling details.
kubectl describe pod <pod-name> -n <namespace>

# Read current container logs. Add -c <container> for multi-container pods.
kubectl logs <pod-name> -n <namespace> --tail=200

# Read logs from the previous crashed/restarted container instance.
kubectl logs <pod-name> -n <namespace> --previous --tail=200

# Follow current logs live.
kubectl logs -f <pod-name> -n <namespace>

# Inspect recent events in time order.
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

Examples using current resource names:

```bash
kubectl logs daemonset/falco -n default --all-pods=true --tail=200
kubectl describe daemonset falco -n default
kubectl logs deployment/trivy-operator -n trivy-system --tail=200
kubectl describe deployment trivy-operator -n trivy-system
kubectl describe statefulset prometheus-monitoring-kube-prometheus-prometheus -n default
```

**Why:** `describe` and events explain scheduling/probe/image failures; logs explain application errors. `--previous` is useful when the current container is running but an earlier instance restarted.

## 5. Check metrics and security findings

```bash
# Confirm Trivy and Falco pods and services exist.
kubectl get pods,svc -n trivy-system -o wide
kubectl get pods,svc -n default | grep -E 'falco|prometheus|grafana|alertmanager'

# List Trivy reports. These report objects contain findings from scans.
kubectl get vulnerabilityreports -A
kubectl get configauditreports -A
kubectl get exposedsecretreports -A
kubectl get clustercompliancereports -A

# List Prometheus targets (run while its port-forward is active).
curl -fsS http://127.0.0.1:19090/api/v1/targets | python3 -m json.tool | less

# Query Trivy severity totals directly in Prometheus.
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="Critical"})'
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="High"})'
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="Medium"})'
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="Low"})'
```

**Why:** Prometheus target health verifies scraping; direct metric queries distinguish missing source data from a dashboard problem. Trivy’s report objects can exist before metrics update. `No data` means the query returned no series; it is not equivalent to zero findings.

### Run the checks from the recent Lens review

This is a copy-ready read-only sequence covering cluster health, workloads, services, Trivy reports, and Prometheus metrics. The Prometheus port-forward must already be running in another terminal for the `curl` commands.

```bash
# Confirm the active cluster and API/node health.
kubectl config current-context
kubectl cluster-info
kubectl get nodes -o wide

# Check all pods, controller readiness, services, and recent cluster events.
kubectl get pods -A -o wide
kubectl get deploy,sts,ds -A
kubectl get services -A
kubectl get events -A --sort-by=.lastTimestamp | tail -50

# Review Trivy scan reports across all namespaces.
kubectl get vulnerabilityreports -A
kubectl get configauditreports -A
kubectl get exposedsecretreports -A
kubectl get clustercompliancereports -A

# Check Prometheus targets in a readable format.
curl -fsS http://127.0.0.1:19090/api/v1/targets | python3 -m json.tool | less

# Get cluster-wide Trivy vulnerability totals by severity.
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="Critical"})'
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="High"})'
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="Medium"})'
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="Low"})'
```

At the time of the terminal check shared on 2026-10-17, Prometheus returned **3 Critical, 429 High, 349 Medium, and 74 Low** across the cluster. These counts change as scans and workloads change; rerun the queries for current values. The Lens detail for `docker.io/falcosecurity/falco:0.44.1` showed an image-specific result of **3 High, 9 Medium, and 5 Low**, scanned about 11 minutes before that screenshot. Image-specific counts and cluster-wide totals have different scopes.

To find the report associated with the Falco image, list reports in `default`, identify the relevant name, then describe it:

```bash
kubectl get vulnerabilityreports -n default
kubectl describe vulnerabilityreport <report-name> -n default
```

The report name varies by pod/image; use the name returned by the first command. Do not treat the screenshot values as current scan results.

## 6. Safe recovery actions

Start with read-only diagnosis. If a deployment needs a restart and you have confirmed that restarting is appropriate:

```bash
# Restart a Deployment and wait for its rollout.
kubectl rollout restart deployment/<deployment-name> -n <namespace>
kubectl rollout status deployment/<deployment-name> -n <namespace> --timeout=180s

# Restart a DaemonSet and wait.
kubectl rollout restart daemonset/<daemonset-name> -n <namespace>
kubectl rollout status daemonset/<daemonset-name> -n <namespace> --timeout=180s

# Review rollout history and revert a bad Deployment rollout if a prior revision exists.
kubectl rollout history deployment/<deployment-name> -n <namespace>
kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

**Why:** A controller recreates managed pods after a restart. This is disruptive to that service while replacement pods start; avoid restarting core control-plane components as a first response. Do not delete pods, services, PVCs, Helm releases, or the kind node casually. For Helm-managed workloads, inspect the release before changing resources:

```bash
helm list -A
helm status monitoring -n default
helm status trivy-operator -n trivy-system
```

## 7. Portainer and Docker boundary

Portainer at `https://localhost:9443` shows Docker containers. In this kind setup, `devsecops-control-plane` is the whole Kubernetes node. Kubernetes workloads such as Grafana, Prometheus, Falco and Trivy are pods **inside** that node, so manage them with `kubectl`, Helm, Grafana, or Kubernetes Dashboard rather than removing Docker containers in Portainer.

Use Docker commands only for read-only inspection unless you deliberately intend to change the cluster:

```bash
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
docker inspect devsecops-control-plane
```

**Why:** Stopping/removing the kind node container can interrupt or destroy the entire cluster and its local state.

## 8. Advanced Docker container dashboard

Open the provisioned Grafana dashboard at [Docker Containers Advanced](http://localhost:13000/d/docker-containers-overview/docker-containers-advanced?orgId=1&refresh=30s). It is adapted from [Grafana Labs dashboard 21743](https://grafana.com/grafana/dashboards/21743-cadvisor-exporter-docker-containers-overview/) to the local `docker_container_*` Prometheus metrics. It provides container totals and state, an inventory, CPU and memory usage, memory-limit percentage, network rates, and block-device I/O. Use the **Container** selector to filter the time-series panels. Portainer is linked from the dashboard for container actions.

The adapted dashboard JSON is saved at `Kubernetes Monitoring/Grafana Dashboards/Docker Containers Advanced.json` in this vault. Kubernetes provisions it from ConfigMap `default/docker-container-grafana-dashboard`; the Grafana dashboard sidecar reloads changes to that ConfigMap.

```bash
# Confirm that the dashboard is provisioned and inspect its panel count.
kubectl get configmap docker-container-grafana-dashboard -n default \
  -o jsonpath='{.data.docker-containers\.json}' | python3 -c \
  'import json,sys; d=json.load(sys.stdin); print(d["title"], "panels:", len(d["panels"]))'

# Check that the Grafana pod and dashboard sidecar are healthy and reloaded it.
kubectl get pods -n default | grep grafana
kubectl logs -n default deployment/monitoring-grafana -c grafana-sc-dashboard --since=10m

# Check that Prometheus has current Docker container metrics.
curl -fsSG http://127.0.0.1:19090/api/v1/query \
  --data-urlencode 'query=count(docker_container_info)'
curl -fsSG http://127.0.0.1:19090/api/v1/query \
  --data-urlencode 'query=docker_container_cpu_percent'
```

**Why:** The catalog dashboard expects cAdvisor metric names that are not emitted for host Docker containers in this setup. The adapted panels use the metrics that Prometheus actually scrapes here. This source does not currently expose container restart counts; use the inventory/state panels for lifecycle visibility.

## 9. Unified DevSecOps dashboard and dashboard health

Use [DevSecOps Control Plane Advanced](http://localhost:13000/d/devsecops-control-plane-advanced/devsecops-control-plane-advanced?orgId=1&refresh=30s) as the main landing page. It combines cluster readiness, pods, Prometheus target health, alerts, Trivy findings, Falco events, Kubernetes resource/network trends, etcd, and host Docker containers. It links to the component dashboards for investigation. [Alertmanager Advanced](http://localhost:13000/d/eea-9_sik/alertmanager-advanced?orgId=1&refresh=30s) replaces HA peer panels with alert delivery, silences, reload health, and latency metrics suited to this single-replica cluster.

The Grafana dashboard variables now default to **All** and use regex matching, so the single cluster and its namespace/pod data appear without a manual selection. Prometheus ServiceMonitor endpoints carry the target label `cluster=kind-devsecops`, which populates the cluster selector. The selected dashboard targets were queried against Prometheus after the change; they returned data. The dashboard JSON exports are in `Kubernetes Monitoring/Grafana Dashboards/` in this vault.

```bash
# Confirm the cluster label is visible to Grafana's cluster selector.
curl -fsS http://127.0.0.1:19090/api/v1/label/cluster/values | python3 -m json.tool

# Check all ServiceMonitors selected by the monitoring release and their endpoint relabelings.
kubectl get servicemonitors -A -l release=monitoring -o json | python3 -c \
  'import json,sys; d=json.load(sys.stdin); [(print(x["metadata"]["namespace"],x["metadata"]["name"], [e.get("relabelings",[]) for e in x["spec"].get("endpoints",[])])) for x in d["items"]]'

# List Grafana dashboard ConfigMaps and confirm the reload sidecar is healthy.
kubectl get configmaps -n default -l grafana_dashboard=1
kubectl logs -n default deployment/monitoring-grafana -c grafana-sc-dashboard --since=10m

# Check cluster, alert, security, and container overview metrics directly.
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(kube_node_status_condition{condition="Ready",status="true"})'
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(ALERTS{alertstate="firing"}) or vector(0)'
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="Critical"})'
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(rate(falcosecurity_falco_rules_matches_total[5m]))'
curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=count(docker_container_info)'
```

The dashboard updates changed Grafana ConfigMaps directly. A later Helm upgrade can replace chart-managed ServiceMonitor relabelings or dashboards; rerun the validation commands afterward. A blank persistent-volume view is expected while the cluster has no PVs or PVCs.

## Related notes

- [[Kind DevSecOps Monitoring Runbook]]
- [[Application and DevSecOps - Index]]
