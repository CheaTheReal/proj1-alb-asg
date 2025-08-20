Highly Available Web App on AWS (ALB + ASG)

A production-style, auto-healing web stack built 100% in the AWS Console. It’s designed for small businesses and teams that want reliable uptime without a big budget or complex tooling.

What this solves (business outcomes)

Stays online if a server fails (no single point of failure).

Zero-touch recovery: failed servers are removed from traffic and automatically replaced.

Safer exposure: only the Load Balancer is public; app servers are private behind it.

Simple to operate: no SSH, no custom images, repeatable with a single launch template.

What’s included (deliverables)

Application Load Balancer (public :80) in 2 Availability Zones

Target Group with HTTP health checks (/)

Auto Scaling Group (2× t2.micro) across 2 AZs with ELB health checks

Launch Template (Amazon Linux 2023) with user data to install Apache and render the instance ID

Least-privilege Security Groups (ALB is public; EC2 only trusts the ALB SG)

Runbook (how to demo auto-healing) and Cleanup (how to avoid charges)

Evidence screenshots (below)

Architecture (how it works)

The ALB is the only public entry point, listening on HTTP :80.

Requests are forwarded to the Target Group, which sends traffic only to healthy instances.

The Auto Scaling Group keeps the desired count (2). If one instance fails:

ALB marks it unhealthy and stops sending traffic.

ASG launches a replacement to get back to 2.

Security Groups enforce that only the ALB may talk to app instances on :80.

Flow:
Internet → ALB :80 → Target Group (health check /) → ASG (2× EC2, 2 AZs)

Screenshots (evidence)
<h3>Security Groups</h3> <img src="./screenshots/01-sg-alb-inbound.png" alt="ALB inbound 0.0.0.0/0 :80" width="900"> <img src="./screenshots/02-sg-ec2-inbound.png" alt="EC2 allows :80 from ALB SG" width="900"> <h3>Target Group</h3> <img src="./screenshots/03-tg-healthcheck.png" alt="Health check path /" width="900"> <img src="./screenshots/04-tg-healthy-targets.png" alt="Healthy targets across two AZs" width="900"> <h3>Load Balancer</h3> <img src="./screenshots/05-alb-description-dns.png" alt="ALB details and DNS name" width="900"> <img src="./screenshots/06-alb-listener.png" alt="Listener :80 forwarding to target group" width="900"> <h3>Auto Scaling Group</h3> <img src="./screenshots/07-asg-activity-replace.png" alt="ASG activity replacing a terminated instance" width="900"> <img src="./screenshots/08-asg-instances-inservice.png" alt="Two instances InService after heal" width="900"> <h3>App proof (refresh shows different IDs)</h3> <img src="./screenshots/09-app-instanceA.png" alt="App served by instance A" width="900"> <img src="./screenshots/10-app-instanceB.png" alt="App served by instance B" width="900">
Tech details (for engineers & reviewers)

Launch Template user data (Amazon Linux 2023):

<pre><code>#!/bin/bash set -euxo pipefail dnf -y update dnf -y install httpd # IMDSv2 token for metadata TOKEN=$(curl -sS -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600") INSTANCE_ID=$(curl -sS -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id) echo "&lt;h1&gt;Hi from $INSTANCE_ID&lt;/h1&gt;" &gt; /var/www/html/index.html systemctl enable httpd systemctl start httpd </code></pre>

Security Groups

proj1-alb-sg: inbound HTTP :80 from 0.0.0.0/0

proj1-ec2-sg: inbound HTTP :80 from ALB security group (SG-as-source)

Health checks (Target Group)

Protocol: HTTP, Path: /, Success codes: 200

How to demo (2-minute runbook)

See: docs/runbook.md

Refresh the ALB DNS to see instance IDs alternate.

Terminate one instance → no downtime; ASG replaces it; ALB only sends traffic to the healthy one.

Cleanup (avoid charges)

See: docs/cleanup.md

Delete in order: ASG → Launch Template → ALB → Target Group → Security Groups.

Cost notes (tiny for demos)

ALB has a small hourly fee.

t2.micro may be Free Tier (depending on your account).
For portfolio demos: capture screenshots/video, then tear down the stack.

Who this is for

Solo founders & small businesses that want simple, reliable uptime.

Early-career cloud/devops folks who need a real HA reference build for interviews.

Teams that want a minimal, auditable starting point before adding HTTPS, domains, alarms, and CI/CD.

Optional next steps (easy add-ons)

HTTPS + domain (ACM certificate + :443 listener, redirect :80→:443)

Path-based routing (e.g., /api* to a second target group on a different port)

CloudWatch alarms (ALB 5xx, UnHealthyHostCount)

Blue/green updates using new Launch Template versions

Repo structure
.
├─ README.md
├─ docs/
│  ├─ runbook.md
│  └─ cleanup.md
└─ screenshots/
   ├─ 01-sg-alb-inbound.png
   ├─ 02-sg-ec2-inbound.png
   ├─ 03-tg-healthcheck.png
   ├─ 04-tg-healthy-targets.png
   ├─ 05-alb-description-dns.png
   ├─ 06-alb-listener.png
   ├─ 07-asg-activity-replace.png
   ├─ 08-asg-instances-inservice.png
   ├─ 09-app-instanceA.png
   └─ 10-app-instanceB.png

License

MIT (optional)
