
```
Internet → ALB only (port 80, 443)
ALB → App EC2 only (port 8080)
App EC2 → RDS only (port 5432)
App EC2 → outbound internet via NAT (for updates)
Bastion → App EC2 only (port 22)
```

