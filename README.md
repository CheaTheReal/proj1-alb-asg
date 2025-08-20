# Project 1 — Highly Available Web App on AWS (ALB + ASG)

Production-style high availability on AWS using only the Console:
- Application Load Balancer (HTTP :80) → Target Group (health check /)
- Auto Scaling Group (2× t2.micro across 2 AZs) via Launch Template
- Least-privilege Security Groups (ALB is public; instances accept :80 only from the ALB SG)
- User data auto-installs Apache and prints the instance ID (so you can see load balancing)

## Screenshots (evidence)

<h3>Security Groups</h3>
<img src="./screenshots/01-sg-alb-inbound.png" alt="ALB inbound" width="900">
<img src="./screenshots/02-sg-ec2-inbound.png" alt="EC2 allows :80 from ALB SG" width="900">

<h3>Target Group</h3>
<img src="./screenshots/03-tg-healthcheck.png" alt="Health check path /" width="900">
<img src="./screenshots/04-tg-healthy-targets.png" alt="Healthy targets across AZs" width="900">

<h3>Load Balancer</h3>
<img src="./screenshots/05-alb-description-dns.png" alt="ALB details + DNS" width="900">
<img src="./screenshots/06-alb-listener.png" alt="Listener :80 → forward to TG" width="900">

<h3>Auto Scaling Group</h3>
<img src="./screenshots/07-asg-activity-replace.png" alt="Replacement after terminate" width="900">
<img src="./screenshots/08-asg-instances-inservice.png" alt="Two instances InService" width="900">

<h3>App proof (refresh shows different IDs)</h3>
<img src="./screenshots/09-app-instanceA.png" alt="Instance A" width="900">
<img src="./screenshots/10-app-instanceB.png" alt="Instance B" width="900">

## Runbook & Cleanup
- Runbook (2-minute chaos test): docs/runbook.md  
- Cleanup (avoid charges): docs/cleanup.md

## What this demonstrates
- Multi-AZ high availability with ALB + ASG
- SG-to-SG least-privilege networking
- Health checks removing bad targets; ASG auto-healing
- Zero-touch bootstrap via Launch Template user data

