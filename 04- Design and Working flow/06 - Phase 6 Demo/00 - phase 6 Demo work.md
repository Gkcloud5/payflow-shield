
```
Demo Website
├── Send 100 jobs to SQS     → queue depth rises
│   → CloudWatch alarm fires → ASG adds workers
├── Stress CPU on workers    → CPU alarm fires
│   → ASG adds more workers
└── Live metrics panel       → shows queue depth
                                + worker count updating
```

```
Demo Architecture
├── S3 Static Website      → hosts the HTML page
├── Lambda API             → talks to SQS + ASG + CloudWatch
└── API Gateway            → connects website to Lambda
```

