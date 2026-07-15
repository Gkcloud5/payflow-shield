
```
Phase 3 Stacks
├── kms.yaml          → Encryption keys for all data
├── cloudtrail.yaml   → Audit log of every AWS action
└── guardduty.yaml    → Intelligent threat detection
```

## Stack 1 — KMS Encryption Keys

```
payflow-shield/
└── infrastructure/
    └── stacks/
        └── kms.yaml    ← create this now
```

### Stack 2 — CloudTrail + S3 Audit Logs

```
payflow-shield/
└── infrastructure/
    └── stacks/
        └── cloudtrail.yaml    ← create this now
```


## Stack 3 — GuardDuty Threat Detection

```
payflow-shield/
└── infrastructure/
    └── stacks/
        └── guardduty.yaml    ← create this now
```