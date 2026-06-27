

### Stack

![[Pasted image 20260627165837.png]]

![[Pasted image 20260627170000.png]]

### What i have build in phase 1:

```
payflow-shield-network-dev
├── VPC (10.0.0.0/16)
├── 2 Public Subnets      → ALB, Bastion
├── 2 Private App Subnets → EC2 Workers (Phase 2)
├── 2 Private DB Subnets  → RDS PostgreSQL
├── Internet Gateway      → public internet access
├── NAT Gateway           → private subnet outbound access
└── Route Tables          → traffic routing rules

payflow-shield-security-dev
├── ALB Security Group    → accepts port 80, 443 from internet
├── App Security Group    → accepts port 8080 from ALB only
├── DB Security Group     → accepts port 5432 from App only
└── Bastion Security Group→ accepts port 22 for SSH

payflow-shield-compute-dev
├── Bastion Host (t3.micro) → secure SSH entry point
└── ALB + Target Group      → load balancer ready for workers

payflow-shield-database-dev
├── RDS PostgreSQL 15.4     → financial-grade database
├── DB Subnet Group         → locked in private subnets
└── Secrets Manager         → auto-generated password, zero plaintext
```

### Key things i did:

* Identified the plaintext password and used secret manager for it