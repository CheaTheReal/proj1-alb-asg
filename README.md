# Project 1 — Highly Available Web App (ALB + ASG)

A tiny production-style web stack on AWS:
- Application Load Balancer → Target Group (health check `/`)
- Auto Scaling Group (2x t2.micro across 2 AZs)
- Launch Template (Amazon Linux 2023 + Apache via user data)
- Security: ALB SG allows :80 from internet; EC2 SG allows :80 **only** from ALB SG

## Screenshots
See the `/screenshots` folder.

## Cleanup
See `/docs/cleanup.md` (to avoid charges).
