# Payflow-shield

```
Imainge thousand of salespeople across many companies every month, someone needs to calculate how much money each salesperson earned as bonus(commision). Payslow shield is used to build the environemnt for that SaSS product run without any issues.
```

## 💡 What Is This?

**PayFlow Shield** is a cloud system built on **Amazon Web Services (AWS)**.

Think of AWS like renting space, computers, and tools from Amazon — without buying any physical hardware.

This project simulates what a **real FinTech (Financial Technology) company** uses to:

- Receive commission data from sales tools (like Salesforce)
- Calculate bonuses for thousands of salespeople
- Store the results safely and securely
- Never lose any data, even if something breaks

> 🏦 **Real companies that use similar systems:** Everstage, Stripe, Razorpay, CaptivateIQ

## 🔥 The Problem It Solves

| ❌ Without PayFlow Shield                         | ✅ With PayFlow Shield                                               |
| ------------------------------------------------ | ------------------------------------------------------------------- |
| Server crashes when too many jobs arrive at once | System automatically adds more workers to handle the load           |
| Company A can accidentally see Company B's data  | Each company's data is completely separated and locked              |
| No record of who did what — auditors unhappy     | Every single action is logged and stored permanently                |
| Database passwords written in plain text in code | Passwords are stored in a secure vault and rotated automatically    |
| If database crashes, data is lost forever        | Automatic backup in a second location — recovers in under 5 minutes |
| Deployment breaks the live system                | New code is tested first, then deployed with zero downtime          |

## 📖 How It Works — Simple Story

Imagine a **restaurant order system** — but for commissions:

```
Step 1️⃣  A salesperson closes a deal in Salesforce
           ↓
Step 2️⃣  Salesforce sends the data to our system via an API
         (API = a door that accepts requests from outside)
           ↓
Step 3️⃣  The request goes into a Queue (like a waiting line at a restaurant)
         Nobody gets overwhelmed — everyone waits their turn
           ↓
Step 4️⃣  Workers (computers) pick up jobs from the queue one by one
         If there are 10,000 jobs, more workers automatically start up
           ↓
Step 5️⃣  Each job is calculated and the result is saved to the Database
         Think of the database as a very organized filing cabinet
           ↓
Step 6️⃣  Everything is logged — who did what, when, and why
         Even AWS admins cannot delete these logs
           ↓
Step 7️⃣  If anything breaks, the system heals itself automatically
         A small robot (Lambda) watches for problems and fixes them
```

## 🛠️ AWS Services Used

> **What is AWS?** Amazon Web Services is like a giant toolbox in the cloud. Instead of buying your own computers and servers, you rent exactly what you need.

|AWS Service|What It Does|Simple Analogy|
|---|---|---|
|**VPC**|Your own private network inside AWS|Like renting your own floor in a building|
|**EC2**|Virtual computers that run your code|Like renting a laptop from Amazon|
|**RDS PostgreSQL**|The database that stores all commission data|A super-organized, automatic filing cabinet|
|**SQS**|A queue that holds jobs waiting to be processed|The waiting line at a restaurant|
|**API Gateway**|The front door that receives requests|A reception desk that routes visitors|
|**Lambda**|Small functions that run automatically|A robot that wakes up, does one job, goes back to sleep|
|**CloudFormation**|Blueprint to build the entire system with one command|Like an IKEA instruction manual for cloud infrastructure|
|**CloudTrail**|Records every single action taken|Like a security camera for your AWS account|
|**KMS**|Encrypts (locks) all your data|A master key vault|
|**S3**|Stores files, logs, and backups|A giant hard drive in the cloud|
|**CloudWatch**|Monitors everything and sends alerts|The dashboard in a car — shows speed, fuel, warnings|
|**CodePipeline**|Automatically deploys new code|An assembly line that tests and ships code|
|**WAF**|Blocks bad traffic before it enters|A security guard at the front door|
|**GuardDuty**|Detects unusual or suspicious activity|A detective watching for suspicious behavior|
|**Secrets Manager**|Stores passwords securely, rotates them automatically|A bank vault for passwords|
|**Auto Scaling Group**|Adds or removes workers based on how busy the system is|Calling in more staff when the restaurant gets busy|

🏗️ Architecture — The Big Picture

```
🌍 OUTSIDE WORLD
               (Salesforce, HubSpot, NetSuite)
                           │
                           ▼
              🛡️  WAF (Security Guard)
                           │
                           ▼
              🚪  API Gateway (Front Door)
                           │
                           ▼
              🔐  Cognito (Login & Identity Check)
                           │
                           ▼
              📬  SQS Queue (Waiting Line)
              📭  Dead Letter Queue (Failed Jobs)
                           │
                           ▼
    ⚙️  EC2 Workers (Auto Scaling Group)
    [Worker] [Worker] [Worker] ← adds more when busy
                           │
              ┌────────────┴───────────┐
              ▼                        ▼
    🗄️  RDS PostgreSQL           ⚡ Lambda Functions
        (Main Database)          (Auto-healing robots)
        Multi-AZ Backup
              │
              ▼
    📦  S3 Storage                📊  Redshift
        Object Lock (Immutable)       (Analytics)
        Glacier (Old Logs)
              │
              ▼
    👁️  OBSERVABILITY LAYER
        CloudWatch | CloudTrail | Config | GuardDuty
              │
              ▼
    🚀  CI/CD PIPELINE
        GitHub → CodePipeline → CodeBuild → CodeDeploy → EC2
```


## 📅 Project Phases

This project is built in **5 phases**, one at a time. Like building a house — foundation first, then walls, then roof.

| Phase                                     | What We Build                                                                | Status         |
| ----------------------------------------- | ---------------------------------------------------------------------------- | -------------- |
| **Phase 1** 🏗️ — Foundation              | VPC Network + CloudFormation Templates + 3 Environments (Dev, Staging, Prod) | 🔄 In Progress |
| **Phase 2** ⚙️ — Processing Engine        | SQS Queue + EC2 Workers + RDS Database + Auto Scaling                        | ⬜ Upcoming     |
| **Phase 3** 🔐 — Security & Compliance    | KMS Encryption + Audit Logs + GuardDuty + Secrets Manager                    | ⬜ Upcoming     |
| **Phase 4** 📊 — Observability & Recovery | CloudWatch Dashboards + Self-Healing + Disaster Recovery Drills              | ⬜ Upcoming     |
| **Phase 5** 🚀 — CI/CD Pipeline           | GitHub + CodePipeline + Blue/Green Deployment + Cost Optimization            | ⬜ Upcoming     |

