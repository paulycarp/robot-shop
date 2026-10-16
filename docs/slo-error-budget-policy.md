# Service Level Objective (SLO) & Error Budget Policy
**Target Application**: Stan's Robot Shop (Core Services: `web`, `catalogue`, `cart`, `payment`)  
**Owner**: DevOps / SRE Team (`paulycarp`)  
**Review Period**: Monthly Rolling Window (30 Days)

---

## 1. Executive Summary & Objective
This document defines the Service Level Indicators (SLIs), Service Level Objectives (SLOs), and Error Budget management framework for the Stan's Robot Shop e-commerce microservices platform. The goal is to provide reliable, high-performing checkout and catalogue experiences while maintaining continuous feature delivery.

---

## 2. SLIs & SLO Target Definitions

| Service | Service Level Indicator (SLI) | SLO Target | Measurement Window |
| :--- | :--- | :--- | :--- |
| **Payment Service** | Proportion of successful checkout transactions (HTTP non-5xx) | **99.5% Availability** | 30-day rolling window |
| **Catalogue & Cart** | Proportion of catalogue queries and cart mutations under 500ms | **99.0% Latency & Uptime** | 30-day rolling window |
| **Web Frontend** | Proportion of successful user page loads (HTTP 200/300 status codes) | **99.0% Availability** | 30-day rolling window |

---

## 3. Error Budget Formulation
The Error Budget represents the total allowable unreliability before user happiness degrades:

$$\text{Error Budget} = 100\% - \text{SLO}$$

* For **99.0% Availability**:
  * Allowable Downtime / Failure per month: $0.01 \times 720 \text{ hours} = \mathbf{7.2\text{ hours}}$ (or $\approx 14.4$ minutes/day).
* For **99.5% Availability (Payment)**:
  * Allowable Downtime / Failure per month: $0.005 \times 720 \text{ hours} = \mathbf{3.6\text{ hours}}$.

---

## 4. Alerting & Burn Rate Policies

Alerts are calculated dynamically by Prometheus and dispatched via Alertmanager:
* **Critical Page (Slack Alert)**: Triggered when burn rate consumes $> 2\%$ of the monthly budget in 1 hour (rapid burn).
* **Warning (Notification)**: Triggered when burn rate consumes $> 5\%$ of the budget in 6 hours.
* **Notification Target**: Dedicated `#devops-alerts` Slack channel via Incoming Webhook.
* **Direct Remediation Link**: Every alert contains an immutable `runbook_url` pointing directly to [Runbook Documentation](runbook.md).

---

## 5. Error Budget Consequences (The SRE Agreement)

| Budget Remaining | Deployment Status | Engineering Action |
| :--- | :--- | :--- |
| **$> 50\%$ Remaining** | Normal Operations | Feature releases and updates proceed as scheduled. |
| **$10\% - 50\%$ Remaining** | Heightened Alert | Pre-deployment canary testing required; non-critical releases reviewed. |
| **$< 10\%$ (Near Exhaustion)** | Feature Freeze | **No new features pushed.** All sprint resources diverted to reliability, automated testing, bug fixes, and infrastructure stabilization. |
| **$0\%$ (Budget Depleted)** | Complete Freeze | Postmortem required; CEO/Lead sign-off required to resume deployments. |

---

## 6. Escalation & Ownership
* **Tier 1 (Automated)**: Prometheus detects breach $\rightarrow$ Alertmanager alerts Slack with linked Runbook.
* **Tier 2 (On-Call Engineer)**: Mitigates within 15 minutes following Runbook steps.
* **Tier 3 (Lead Engineer)**: Root-cause identification and remediation.
* **Incident Close**: Blameless Postmortem published within 48 hours.
