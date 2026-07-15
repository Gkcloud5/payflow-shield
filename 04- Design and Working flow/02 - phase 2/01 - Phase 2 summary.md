
```
payflow-shield-sqs-dev
├── CommissionQueue        → receives all incoming jobs
│   ├── VisibilityTimeout  → 300 seconds per job
│   ├── RetentionPeriod    → 24 hours
│   └── MaxReceiveCount    → 3 attempts before DLQ
└── CommissionDLQ          → catches all failed jobs
    └── RetentionPeriod    → 14 days for investigation

payflow-shield-iam-dev
├── WorkerRole             → IAM role for EC2 workers
├── WorkerPolicy           → least privilege permissions
│   ├── SQS                → read + delete messages only
│   ├── Secrets Manager    → fetch DB password only
│   └── CloudWatch         → write logs + metrics only
└── WorkerInstanceProfile  → attaches role to EC2

payflow-shield-workers-dev
├── LaunchTemplate         → worker EC2 configuration
│   ├── AMI                → Amazon Linux 2023
│   ├── Instance Type      → t3.micro
│   ├── Key Pair           → test-iam-new
│   └── UserData           → bootstrap script on startup
├── Auto Scaling Group     → manages worker fleet
│   ├── Min: 1 | Max: 4 | Desired: 1
│   └── Spans 2 private app subnets across 2 AZs
├── ScaleOutPolicy         → adds 1 worker when queue > 10
└── ScaleInPolicy          → removes 1 worker when queue < 1
```

```
Commission job arrives
        ↓
CommissionQueue receives it
        ↓
CloudWatch sees queue depth > 10
        ↓
ScaleOutAlarm fires → ASG adds workers
        ↓
Workers pick up jobs → process → write to RDS
        ↓
Job fails 3 times → moves to DLQ
        ↓
Queue empties → ScaleInAlarm fires → ASG removes workers
```

