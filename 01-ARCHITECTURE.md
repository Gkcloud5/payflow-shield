
# 🛡️ PayFlow Shield — Architecture

> **Type:** AWS Portfolio Project · Production-Grade FinTech SaaS  
> **Region:** ap-south-1 (Mumbai)  
> **Goal:** Multi-Tenant Commission Processing Platform  
> **Timeline:** 1 Week · 5 Phases


## 🗺️ Tags

`#aws` `#payflow-shield` `#architecture` `#cloud-engineering` `#fintech`

---

## 🔄 Commission Job — Data Flow

```
1. CRM (Salesforce) sends commission job via HTTPS
        ↓
2. WAF inspects → API Gateway receives → Cognito authenticates tenant
        ↓
3. Job placed into SQS Standard Queue
        ↓
4. ASG detects queue depth rising → spins up EC2 workers
        ↓
5. EC2 Worker picks up job → calculates commission
        ↓
6. Result written to RDS PostgreSQL (with Row-Level Security)
        ↓
7. Data encrypted by KMS → audit log written to S3 Object Lock
        ↓
8. CloudWatch monitors → GuardDuty watches → SNS alerts if anomaly
        ↓
9. Entire infra deployed and updated via CodePipeline + CloudFormation
```


## 📌 What is PayFlow Shield?

A production-grade AWS infrastructure that simulates a real FinTech SaaS platform — like **Everstage**, **CaptivateIQ**, or **Spiff**.

It processes **commission calculations** for thousands of salespeople across multiple companies — with:

- ✅ Zero tolerance for data loss
- ✅ Financial-grade audit compliance (SOC 2)
- ✅ Multi-tenant data isolation
- ✅ Auto-scaling under month-end compute spikes
- ✅ Full Infrastructure as Code (no console clicks)

---

## 🏗️ Full Architecture — Layer by Layer

### Layer 1 — 🌐 External Integration

> CRM systems that send commission data into the platform

| System     | Role                      |
| ---------- | ------------------------- |
| Salesforce | Primary CRM source        |
| HubSpot    | Secondary CRM source      |
| NetSuite   | ERP / finance data        |
| REST APIs  | Standard HTTP integration |

---

### Layer 2 — 🛡️ Security Edge

> First line of defence — blocks bad traffic, authenticates tenants

| Service                             | Role                                                  |
| ----------------------------------- | ----------------------------------------------------- |
| **WAF** (Web Application Firewall)  | Blocks malicious HTTP requests                        |
| **API Gateway**                     | Entry point for all commission job requests           |
| **Cognito**                         | SSO authentication — verifies which tenant is calling |
| **Application Load Balancer (ALB)** | Distributes traffic to EC2 workers                    |

---

### Layer 3 — 🏗️ VPC Network _(Phase 1)_

> Your private network inside AWS — controls every door and corridor

```
VPC: Custom (not default)
│
├── Public Subnets      → AZ-a  |  AZ-b   (Bastion Host, NAT Gateway, ALB)
├── Private App Subnets → AZ-a  |  AZ-b   (EC2 Workers)
└── Private DB Subnets  → AZ-a  |  AZ-b   (RDS PostgreSQL)
```

|Component|Purpose|
|---|---|
|**Custom VPC**|Isolated private network|
|**Public Subnets** (×2 AZs)|Face the internet — ALB lives here|
|**Private App Subnets** (×2 AZs)|EC2 workers — no direct internet access|
|**Private DB Subnets** (×2 AZs)|Database — fully isolated|
|**Internet Gateway**|Connects VPC to the internet|
|**NAT Gateway**|Lets private subnets reach internet (one-way)|
|**Bastion Host**|Secure SSH jump server|

> 💡 **Why 2 AZs?** AZs are physically separate data centres. If one fails, the system keeps running in the other. This is how 99.99% uptime is achieved.

---

### Layer 4 — 📬 Job Queue _(Phase 2)_

> Absorbs commission job spikes — prevents system overload

|Service|Role|
|---|---|
|**SQS Standard Queue**|Buffers all incoming commission jobs|
|**Dead Letter Queue (DLQ)**|Catches jobs that fail 3+ times|
|**EventBridge**|Triggers batch jobs at month-end automatically|
|**SNS**|Fires alert when DLQ receives failed jobs|

> 💡 **Analogy:** SQS is like a restaurant ticket system. 10,000 orders come in — they don't all hit the kitchen at once. Workers pick them up one by one at their own pace.

---

### Layer 5 — ⚙️ Compute / Workers _(Phase 2 + Phase 4)_

> Auto-scaling EC2 fleet that processes commission calculations

|Service|Role|
|---|---|
|**EC2 Worker Fleet**|Runs the commission calculation code|
|**Auto Scaling Group (ASG)**|Scales workers up/down based on SQS queue depth|
|**Spot Instances**|Up to 70% cost savings vs On-Demand|
|**Lambda**|Self-healing — auto-reboots failed workers|
|**Security Group Chains**|ALB SG → App SG → DB SG (firewall rules)|

> 💡 **Key Pattern:** Scale on **SQS queue depth**, not CPU.  
> 1,000 messages waiting → spin up 100 workers. Queue empty → scale back to 2.

---

### Layer 6 — 🗄️ Data Layer _(Phase 2)_

> Stores commission results — financial-grade reliability

|Service|Role|
|---|---|
|**RDS PostgreSQL (Multi-AZ)**|Primary commission database|
|**Row-Level Security (RLS)**|Tenant isolation at DB level — Company A cannot see Company B|
|**Multi-AZ Standby Replica**|Auto-failover in 60–120 seconds if primary fails|
|**Redshift**|Analytics — run reports across all commission data|
|**ElastiCache (Redis)**|Caching — fast lookups for repeated queries|

> 💡 **Why PostgreSQL?** Supports Row-Level Security (RLS) — enforces tenant isolation at the database layer, even if application code has a bug.

---

### Layer 7 — 🔒 Storage + Compliance _(Phase 3)_

> Immutable audit logs, encrypted backups, financial archival

|Service|Role|
|---|---|
|**S3**|Stores audit logs and DB backups|
|**S3 Object Lock (WORM)**|Nobody can delete or modify audit logs — ever|
|**S3 Glacier**|Logs older than 90 days move to cheap archival automatically|
|**KMS (Customer Managed Keys)**|Encrypts all data at rest — you own the keys|
|**Secrets Manager**|Stores DB passwords — auto-rotates every 30 days|

> 💡 **WORM = Write Once, Read Many.** Like permanent ink sealed in a vault. Required for SOC 2 compliance.

---

### Layer 8 — 📡 Observability + Security _(Phase 3 + Phase 4)_

> Monitor everything, detect threats, audit every action

|Service|Role|
|---|---|
|**CloudWatch Dashboards**|Real-time view of queue depth, EC2 CPU, DB connections, error rates|
|**CloudTrail**|Logs every single AWS API call — who did what, when, from where|
|**AWS Config**|Detects compliance violations (e.g. port 22 open to world)|
|**GuardDuty**|AI threat detection — unusual logins, crypto mining attempts|
|**SNS**|Email/SMS alerts when any threshold is breached|
|**Custom Metrics**|Commission job success rate, processing time, DLQ count|

> **CloudTrail vs AWS Config — Quick Difference:**

||CloudTrail|AWS Config|
|---|---|---|
|Tracks|WHO did WHAT action and WHEN|What a resource LOOKS LIKE right now|
|Example|"User X deleted SG Y at 3pm"|"SG Y has port 22 open to 0.0.0.0/0"|
|Use case|Forensics, audit trail|Compliance checking, drift detection|

---

### Layer 9 — 🚀 CI/CD Pipeline _(Phase 5)_

> Deploy infrastructure and code automatically — zero manual steps

|Service|Role|
|---|---|
|**GitHub**|Single source of truth for all code and IaC templates|
|**CodePipeline**|Detects git push → orchestrates the full pipeline|
|**CodeBuild**|Runs tests, lints CloudFormation, builds application artifact|
|**CodeDeploy**|Blue/Green deployment to EC2 workers — zero downtime|
|**CloudFormation Nested Stacks**|Deploys entire infrastructure as code|
|**Dev / Staging / Prod**|Same template, three environments (dev.json, staging.json, prod.json)|

> 💡 **Blue/Green Deploy:**  
> Blue = current live fleet (v1.0) → Green = new fleet (v1.1)  
> ALB routes 10% traffic to Green → if healthy → 100% to Green → Blue terminated.  
> If errors → instantly switch back. **Zero downtime. Instant rollback.**





