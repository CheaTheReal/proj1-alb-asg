# Highly Available Web App on AWS (ALB + ASG)

Production-style **high availability** on AWS using only the **Console**:
- **Application Load Balancer (HTTP :80)** → **Target Group** (health check `/`)
- **Auto Scaling Group** (**2× t2.micro** across **2 AZs**) via **Launch Template**
- **Least-privilege Security Groups** (ALB is public; instances accept :80 **only** from the ALB SG)
- **User data** auto-installs Apache and prints the **instance ID** (see load balancing)

> **Goal:** One public door (ALB), multiple private servers, auto-healing and health checks.

## Architecture
Internet
|
v
[ Application Load Balancer :80 ]
| sg: proj1-alb-sg (HTTP 80 from 0.0.0.0/0)
v
[ Target Group ] <-- Health check: HTTP GET "/"
|
v
[ Auto Scaling Group (desired=2) across AZ-a / AZ-b ]
| Launch Template: Amazon Linux 2023 + user data
+--> [ EC2 #1 ] sg: proj1-ec2-sg (HTTP 80 from ALB SG only)
+--> [ EC2 #2 ] sg: proj1-ec2-sg

pgsql
Copy
Edit

## What I Practiced / Learned
- Multi-AZ HA with **ALB + ASG**
- **SG-to-SG** least-privilege rules
- TG health checks & how ALB detaches unhealthy targets
- Launch Templates + user data on Amazon Linux 2023
- Using **IMDSv2** (token) inside user data

**User data used:**
```bash
#!/bin/bash
set -euxo pipefail
dnf -y update
dnf -y install httpd
TOKEN=$(curl -sS -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
INSTANCE_ID=$(curl -sS -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)
echo "<h1>Hi from $INSTANCE_ID</h1>" > /var/www/html/index.html
systemctl enable httpd
systemctl start httpd
Screenshots
Security Groups


Target Group


Load Balancer


Auto Scaling Group


App proof (refresh shows different IDs)


Runbook & Cleanup
Runbook: docs/runbook.md

Cleanup (avoid charges): docs/cleanup.md

Real-world uses
Public sites, internal tools that need HA, and a starter pattern before adding HTTPS/domain/alarms/CI/CD.

Next steps
HTTPS + domain (ACM + :443), path-based routing (/api*), CloudWatch alarms, blue/green via LT versions.

Cost notes
ALB has a small hourly fee; t2.micro may be Free Tier. For demos, take screenshots then tear down.

License
MIT (optional)
