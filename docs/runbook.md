# Operational Incident Runbook: Robot Shop Service Degradation
**Alert**: `RobotShopServiceDegraded`  
**Severity**: Critical  
**Target Services**: `web`, `catalogue`, `cart`, `payment`

---

## ⚡ Step 1: Immediate Triage (0–5 Minutes)

### 1.1 Verify Cluster & Pod Status
Run the following command to check if any pods are crashing, restarting, or failing liveness/readiness probes:
```bash
kubectl get pods -n robot-shop -o wide
```
Look for:
* Statuses like `CrashLoopBackOff`, `Error`, or `OOMKilled`.
* Container restart count increasing.

### 1.2 Inspect Recent Kubernetes Events
```bash
kubectl get events -n robot-shop --sort-by='.metadata.creationTimestamp' | tail -n 20
```

---

## 🔍 Step 2: Investigation & Root Cause Identification (5–15 Minutes)

### 2.1 Query Centralized Logs via Grafana Loki
1. Open Grafana: [http://localhost:3000](http://localhost:3000) (Login: `admin` / `admin`).
2. Go to **Explore** and select the **Loki** datasource.
3. Query the affected service for error logs:
   ```logql
   {namespace="robot-shop", container="payment"} |= "error"
   ```
   *(Replace `"payment"` with `"cart"`, `"catalogue"`, or `"web"` depending on the alert).*

### 2.2 Trace the Request Path in Jaeger
1. Open Jaeger UI: [http://localhost:16686](http://localhost:16686) (or port-forward: `kubectl port-forward -n monitoring svc/jaeger 16686:16686`).
2. Select the service from the dropdown and click **Find Traces**.
3. Look for traces with red error badges to pinpoint exactly which database or downstream call timed out.

---

## 🛠️ Step 3: Mitigation Playbook (Stop the Bleed)

### Scenario A: Service Pod is Stuck or Unresponsive
Restart the deployment to restore availability immediately:
```bash
kubectl rollout restart deployment <service-name> -n robot-shop
```

### Scenario B: Bad Deployment / Configuration Push
If a recent deployment or configuration caused the outage, roll back immediately:
```bash
helm rollback robot-shop 1 -n robot-shop
```

### Scenario C: Backend Data Store Disconnected
Check data store health:
```bash
kubectl get pods -l 'service in (mongodb,redis,mysql,rabbitmq)' -n robot-shop
```
If a database is crashing, inspect its storage volume and restart:
```bash
kubectl logs -l service=mysql -n robot-shop --tail=50
```

---

## 📝 Step 4: Resolution & Postmortem Requirements
1. Confirm the alert has resolved in Alertmanager and the Slack channel is updated.
2. Verify Grafana Golden Signals dashboard shows error rate $< 1\%$.
3. Initiate a [Blameless Postmortem](blameless-postmortem.md) within 24 hours.
