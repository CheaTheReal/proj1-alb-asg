Highly Available Web App on AWS (ALB + ASG)

A small, production-style, auto-healing web stack built entirely in the AWS Console — designed for small teams and solo founders who want reliable uptime without heavy ops overhead.

Outcome: If a server fails, traffic keeps flowing and a replacement is created automatically. Instances are private; only the Load Balancer is public.

Quick Links

▶️ Runbook (2-minute demo): docs/runbook.md

🧹 Cleanup (avoid charges): docs/cleanup.md

What This Delivers (Customer Value)

High Availability: Two instances across 2 AZs behind an Application Load Balancer (ALB).

Auto-Healing: Unhealthy instances are removed from traffic and replaced automatically by the Auto Scaling Group (ASG).

Safer Exposure: Only the ALB is public. Instances accept port 80 only from the ALB’s security group (SG-to-SG rule).

Simple Operations: Repeatable bootstrapping via a Launch Template + user data; no SSH required.

Architecture (How It Works)

ALB (HTTP :80) is the single public entry point.

Target Group performs HTTP health checks on /.

ASG (desired=2, min=2, max=2) launches instances across two AZs using a Launch Template.

Security Groups:

proj1-alb-sg → inbound :80 from 0.0.0.0/0

proj1-ec2-sg → inbound :80 from ALB security group (SG-as-source)

<pre><code>Internet │ ▼ [ Application Load Balancer :80 ] (sg: proj1-alb-sg, inbound 0.0.0.0/0) │ ▼ [ Target Group ] -- Health check: HTTP GET "/" │ ▼ [ Auto Scaling Group (2x EC2, 2 AZs) ] ├── [ EC2 #1 ] (sg: proj1-ec2-sg, allows :80 only from ALB SG) └── [ EC2 #2 ] (sg: proj1-ec2-sg) </code></pre>
Implementation Details (For Reviewers)

Launch Template – User Data (Amazon Linux 2023):

<pre><code>#!/bin/bash set -euxo pipefail dnf -y update dnf -y install httpd # IMDSv2 token for metadata TOKEN=$(curl -sS -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600") INSTANCE_ID=$(curl -sS -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id) echo "&lt;h1&gt;Hi from $INSTANCE_ID&lt;/h1&gt;" &gt; /var/www/html/index.html systemctl enable httpd systemctl start httpd </code></pre>

Target Group Health Check

Protocol: HTTP

Path: /

Success codes: 200

Evidence (Screenshots)
<h3>Security Groups</h3> <img src="./screenshots/01-sg-alb-inbound.png" alt="ALB inbound 0.0.0.0/0 :80" width="900"> <img src="./screenshots/02-sg-ec2-inbound.png" alt="EC2 allows :80 from ALB SG" width="900"> <h3>Target Group</h3> <img src="./screenshots/03-tg-healthcheck.png" alt="Health check path /" width="900"> <img src="./screenshots/04-tg-healthy-targets.png" alt="Healthy targets across two AZs" width="900"> <h3>Load Balancer</h3> <img src="./screenshots/05-alb-description-dns.png" alt="ALB details and DNS name" width="900"> <img src="./screenshots/06-alb-listener.png" alt="Listener :80 forwarding to target group" width="900"> <h3>Auto Scaling Group</h3> <img src="./screenshots/07-asg-activity-replace.png" alt="ASG activity replacing a terminated instance" width="900"> <img src="./screenshots/08-asg-instances-inservice.png" alt="Two instances InService after heal" width="900"> <h3>App Proof (refresh shows different IDs)</h3> <img src="./screenshots/09-app-instanceA.png" alt="App served by instance A" width="900"> <img src="./screenshots/10-app-instanceB.png" alt="App served by instance B" width="900">
Demo & Operation

How to demo (2 minutes): docs/runbook.md

Refresh the ALB DNS to watch the instance IDs alternate; terminate one instance to see auto-healing without downtime.

Safe cleanup (stop costs): docs/cleanup.md

Delete in order → ASG → Launch Template → ALB → Target Group → Security Groups.

Cost Considerations (For Stakeholders)

ALB has a small hourly fee; t2.micro may be Free Tier (depending on account age/eligibility).

For demos, record screenshots/video and tear down the stack after use to keep the bill at pennies.

Who This Is For

Small businesses/creators needing reliable uptime without heavy ops.

Early-career cloud engineers showcasing real HA patterns (great portfolio item).

Teams starting simple, then growing into HTTPS, alarms, and CI/CD.

Roadmap (Simple Add-Ons)

HTTPS + custom domain: ACM certificate + :443 listener, redirect :80→:443

Path-based routing: /api* to a second target group (different port)

Observability: CloudWatch alarms (ALB 5xx, UnHealthyHostCount), basic dashboards

Blue/Green: new Launch Template versions with rolling updates

Repository Structure
<pre><code>. ├─ README.md ├─ docs/ │ ├─ runbook.md │ └─ cleanup.md └─ screenshots/ ├─ 01-sg-alb-inbound.png ├─ 02-sg-ec2-inbound.png ├─ 03-tg-healthcheck.png ├─ 04-tg-healthy-targets.png ├─ 05-alb-description-dns.png ├─ 06-alb-listener.png ├─ 07-asg-activity-replace.png ├─ 08-asg-instances-inservice.png ├─ 09-app-instanceA.png └─ 10-app-instanceB.png </code></pre>
License

MIT (optional)
