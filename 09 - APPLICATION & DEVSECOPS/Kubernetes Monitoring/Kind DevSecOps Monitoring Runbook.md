---
title: Kind DevSecOps Monitoring Runbook
category: Operations Runbook
tags:
  - Kubernetes
  - kind
  - Monitoring
  - Troubleshooting
date_created: 2026-10-17
status: ACTIVE
---

# Kind DevSecOps Monitoring Runbook

Use this procedure when a local monitoring URL refuses the connection, a dashboard shows no data, a pod restarts, or Kubernetes Dashboard rejects a token.

Detailed command explanations: [[Kind DevSecOps Monitoring Playbook]].

## A. Start-of-session health check

```bash
kubectl config current-context
kubectl get nodes -o wide
kubectl get pods -A
kubectl get services -A
```

1. Confirm the context is `kind-devsecops` before running commands that modify anything.
2. Confirm the node is `Ready`.
3. Look for pods whose `STATUS` is not `Running`/`Completed`, or whose `READY` count is below the expected count.
4. Treat restart counts as historical clues, not proof of a current outage. Inspect recent events and `--previous` logs for the affected pod.

## B. `localhost:13000` or another UI refuses the connection

1. Confirm the target pod and service:

   ```bash
   kubectl get pods -n default -l app.kubernetes.io/name=grafana
   kubectl get svc monitoring-grafana -n default
   ```

2. If the pod is not ready, inspect it before changing anything:

   ```bash
   kubectl describe pod -n default -l app.kubernetes.io/name=grafana
   kubectl logs deployment/monitoring-grafana -n default --all-containers=true --tail=200
   kubectl get events -n default --sort-by=.lastTimestamp | tail -30
   ```

3. If the pod is ready but the URL refuses the connection, start the local forward and keep the terminal open:

   ```bash
   kubectl -n default port-forward svc/monitoring-grafana 13000:80 --address 127.0.0.1
   ```

4. In a second terminal, verify Grafana:

   ```bash
   curl -fsS http://127.0.0.1:13000/api/health
   ```

5. Open `http://localhost:13000`. The same procedure applies to Prometheus (`19090:9090`), Alertmanager (`19093:9093`) and Kubernetes Dashboard (`8443:443`), using the service/namespace mappings in the playbook.

**Expected:** Grafana health JSON includes `"database": "ok"`. A port-forward process must remain running. If using a remote shell, `localhost` means the remote machine unless you configure SSH tunneling; browser and port-forward need to refer to the same host.

## C. Grafana panel says “No data”

1. Confirm the Trivy Operator and metrics target are available:

   ```bash
   kubectl get pods,svc -n trivy-system -o wide
   kubectl get servicemonitor -A
   ```

2. Start the Prometheus port-forward:

   ```bash
   kubectl -n default port-forward svc/monitoring-kube-prometheus-prometheus 19090:9090 --address 127.0.0.1
   ```

3. Check `http://localhost:19090/targets` for the `trivy-operator` target and confirm it is `UP`.
4. Query a metric directly:

   ```bash
   curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="Critical"})'
   ```

5. Interpret the result:
   - `"status":"success"` with a non-empty `data.result`: Prometheus has data; inspect the Grafana datasource, dashboard query, time range, and panel settings.
   - An empty `data.result`: the metric/label combination currently has no series. Check the report objects and operator scrape target.
   - HTTP error or target down: inspect the ServiceMonitor, service endpoint, and Trivy Operator logs before changing configuration.
6. Verify scan reports:

   ```bash
   kubectl get vulnerabilityreports -A
   kubectl get configauditreports -A
   kubectl get exposedsecretreports -A
   kubectl get clustercompliancereports -A
   ```

7. To check the same cluster-wide totals that were validated from the terminal, run all four queries (the Prometheus forward from step 2 must still be open):

   ```bash
   curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="Critical"})'
   curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="High"})'
   curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="Medium"})'
   curl -fsSG http://127.0.0.1:19090/api/v1/query --data-urlencode 'query=sum(trivy_image_vulnerabilities{severity="Low"})'
   ```

   The last verified sample was 3 Critical, 429 High, 349 Medium, and 74 Low. This is a time-specific cluster-wide snapshot; values can change. Lens showed Falco image `docker.io/falcosecurity/falco:0.44.1` separately with 3 High, 9 Medium, and 5 Low at the time of its scan. For the underlying report, run `kubectl get vulnerabilityreports -n default`, then `kubectl describe vulnerabilityreport <report-name> -n default` with the listed report name.

Do not treat `No data` as a zero count. Check Grafana’s time range and refresh after metrics have been scraped.

## D. Pod is restarting, pending, or not ready

1. Identify namespace, pod name, readiness, restart count and age:

   ```bash
   kubectl get pods -A -o wide
   ```

2. Describe the affected pod and namespace:

   ```bash
   kubectl describe pod <pod-name> -n <namespace>
   kubectl get events -n <namespace> --sort-by=.lastTimestamp | tail -50
   ```

3. Read current and previous logs:

   ```bash
   kubectl logs <pod-name> -n <namespace> --all-containers=true --tail=200
   kubectl logs <pod-name> -n <namespace> --all-containers=true --previous --tail=200
   ```

4. Inspect the owning workload and its rollout:

   ```bash
   kubectl get deploy,sts,ds -n <namespace>
   kubectl describe deployment/<deployment-name> -n <namespace>
   kubectl rollout status deployment/<deployment-name> -n <namespace>
   ```

5. Correct the reported cause first (for example: invalid config, missing image, failed probe, scheduling/capacity issue). Restart only after diagnosis; see recovery commands in the playbook.

Common meanings: `CrashLoopBackOff` means the process repeatedly exits; `ImagePullBackOff` means the image cannot be obtained; `Pending` often means scheduling/storage constraints; `Running` with readiness below target means at least one container is not ready.

## E. Kubernetes Dashboard says “Unauthorized (401)”

1. Check that the intended ServiceAccount exists and has the expected binding:

   ```bash
   kubectl get sa admin-user -n kubernetes-dashboard
   kubectl get clusterrolebinding admin-user
   ```

2. Generate a fresh one-hour token and paste it directly into Dashboard’s Token field:

   ```bash
   kubectl create token admin-user -n kubernetes-dashboard --duration=1h
   ```

3. Keep the token private. Do not add it to notes, scripts, screenshots, or shell history. Generate another when it expires.
4. Ensure the UI port-forward is active:

   ```bash
   kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard 8443:443 --address 127.0.0.1
   ```

The lab’s `admin-user` is bound to `cluster-admin`; only use this broad account for the local lab. Do not expose the Dashboard port publicly.

## F. After recovery

```bash
kubectl get nodes
kubectl get pods -A
kubectl get events -A --sort-by=.lastTimestamp | tail -30
```

Confirm the affected pods are ready, UI health endpoint responds, and relevant Prometheus targets are `UP`. Record the symptom, affected namespace/workload, evidence from events/logs, action taken, and verification result in an incident note. Avoid recording bearer tokens or passwords.

## Related notes

- [[Kind DevSecOps Monitoring Playbook]]
- [[Application and DevSecOps - Index]]
