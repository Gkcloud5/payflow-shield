
```
Phase 2 — Commission Processing Engine

├── SQS Queue + Dead Letter Queue
├── EC2 Worker Fleet (Auto Scaling Group)
├── Workers auto-scale on SQS queue depth
└── Workers read jobs → write results to RDS
```

# About SQS:

* It is managed message queue.
	* Messages go in --> worker picks them up --> Done
* In real world they used to take care
	* Message go in --> Worker might fail --> What happen then? --> Who handles poison message? --> How do we alert on call? --> How do we replay without data loss?
	* That failure ending gap will be resolved by this service

### How A Real Company Uses SQS + DLQ:

```
[Event Happens]
    │
    ▼
[SQS Main Queue]  ← Orders, Payments, Notifications land here
    │
    ▼
[Worker / Lambda]  ← Tries to process the message
    │
    ├── ✅ Success → Message deleted from queue. Done.
    │
    └── ❌ Fails → Message becomes invisible, then reappears
              │
              └── Retried N times (maxReceiveCount)
                        │
                        └── Still failing? → Sent to DLQ
                                    │
                                    └── CloudWatch Alarm fires
                                              │
                                              └── On-call engineer gets paged
```


