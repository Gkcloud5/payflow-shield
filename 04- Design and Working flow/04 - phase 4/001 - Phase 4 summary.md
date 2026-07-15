
```
payflow-shield-cloudwatch-dev
├── CloudWatch Dashboard      → 6 live metric widgets
│   ├── SQS Queue Depth
│   ├── DLQ Message Count
│   ├── Worker EC2 CPU
│   ├── RDS Connections
│   ├── RDS CPU
│   └── ALB Request Count
├── DLQ Alarm                 → fires when failed jobs appear
├── RDS High CPU Alarm        → fires above 80% CPU
├── Worker High CPU Alarm     → fires above 80% CPU
└── Log Groups                → workers + lambda logs

payflow-shield-lambda-dev
├── Self-Healing Lambda       → runs every 5 minutes
│   ├── Checks queue depth
│   ├── Checks healthy worker count
│   └── Auto-scales ASG if workers are down
└── EventBridge Schedule      → triggers Lambda automatically
```

