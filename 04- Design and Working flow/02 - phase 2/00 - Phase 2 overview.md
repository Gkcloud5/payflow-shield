
```
Phase 2 — Commission Processing Engine

├── SQS Queue + Dead Letter Queue
├── EC2 Worker Fleet (Auto Scaling Group)
├── Workers auto-scale on SQS queue depth
└── Workers read jobs → write results to RDS
```

