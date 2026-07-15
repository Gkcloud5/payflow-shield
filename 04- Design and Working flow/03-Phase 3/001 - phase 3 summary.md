
```
payflow-shield-kms-dev
├── Customer Managed KMS Key    → encrypts all data at rest
├── Automatic Key Rotation      → rotates annually, zero downtime
└── Key Alias                   → alias/payflow-shield-dev-key

payflow-shield-cloudtrail-dev
├── CloudTrail                  → records every AWS API call
├── S3 Audit Bucket             → stores all logs encrypted
│   ├── Object Lock (WORM)      → nobody can delete logs for 90 days
│   ├── KMS Encryption          → logs encrypted with your key
│   ├── Versioning              → every log version preserved
│   └── Lifecycle Policy        → moves to Glacier after 90 days
└── Log File Validation         → detects if logs are tampered with

payflow-shield-guardduty-dev
├── SNS Security Alerts Topic   → encrypted with KMS
├── Email Subscription          → alerts land in your inbox
└── EventBridge Rule            → fires alert on root account login
    (GuardDuty → documented as pending account upgrade)
```

