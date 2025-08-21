# **Project 1 — Highly Available Web App on AWS (ALB + ASG)**

<p>
  <img alt="AWS" src="https://img.shields.io/badge/Cloud-AWS-orange.svg">
  <img alt="License" src="https://img.shields.io/badge/License-MIT-lightgrey.svg">
</p>

**Audience:** hiring managers, small-business clients, junior engineers  
**Promise:** a small, production-style, **auto-healing** web stack built **100% in the AWS Console** — clear, auditable, and easy to demo.

---

## **Table of Contents**
1. [TL;DR — What This Does](#tldr--what-this-does)
2. [Why This Matters (Business Outcomes)](#why-this-matters-business-outcomes)
3. [What’s Included (Deliverables)](#whats-included-deliverables)
4. [How Traffic Flows (Step-by-Step)](#how-traffic-flows-step-by-step)
5. [Architecture (At a Glance)](#architecture-at-a-glance)
6. [Implementation Details (Reviewer-Friendly)](#implementation-details-reviewer-friendly)
7. [Evidence (Screenshots)](#evidence-screenshots)
8. [Demo (2 Minutes)](#demo-2-minutes)
9. [Cleanup (Avoid Charges)](#cleanup-avoid-charges)
10. [Cost Notes](#cost-notes)
11. [What I Practiced / Learned](#what-i-practiced--learned)
12. [Who This Helps](#who-this-helps)
13. [Roadmap / V2 Upgrades](#roadmap--v2-upgrades)
14. [Repository Structure](#repository-structure)

---

## **TL;DR — What This Does**
- **One public door:** the **Application Load Balancer (ALB)** is the only thing exposed to the internet.  
- **Two servers in two data centers:** the **Auto Scaling Group (ASG)** keeps **2 EC2 instances** alive across **two Availability Zones**.  
- **If one breaks:** the **ALB** stops sending it traffic and the **ASG** **creates a new one** automatically.  
- **Locked-down access:** servers do **not** accept traffic from the internet; they accept **port 80 only from the ALB** (security-group to security-group).

---

## **Why This Matters (Business Outcomes)**
- **Stays online:** one server can die and the site keeps running.  
- **Low-touch operations:** no SSH needed; instances bootstrap themselves with **User Data**.  
- **Safer by default:** public traffic hits the ALB, not the servers.  
- **Easy to hand off:** screenshots + runbook + cleanup give clients confidence.

---

## **What’s Included (Deliverables)**
- **ALB** (HTTP **:80**) across **2 AZs**  
- **Target Group** (health check **`/`**)  
- **ASG** (**2× t2.micro**, min=2, max=2) with **ELB health checks**  
- **Launch Template** (Amazon Linux 2023) + **User Data** (installs Apache, prints **instance ID**)  
- **Security Groups:**  
  - **`proj1-alb-sg`** → inbound **80** from **0.0.0.0/0**  
  - **`proj1-ec2-sg`** → inbound **80** **only from** `proj1-alb-sg` (SG-as-source)  
- **Docs:** short **Runbook** and **Cleanup**  
- **Evidence:** screenshots below

---

## **How Traffic Flows (Step-by-Step)**
1. **User** requests your ALB DNS name.  
2. **ALB** checks which servers are **healthy** in the **Target Group**.  
3. **ALB** forwards the request to a healthy server.  
4. If a server stops responding, **ALB** marks it unhealthy and stops sending traffic.  
5. **ASG** notices you’re below desired capacity and **launches a replacement**.  
6. When the new server passes health checks, it starts serving traffic.

---

## **Architecture (At a Glance)**
<pre><code>Internet
   │
   ▼
[ Application Load Balancer :80 ]
   │  (sg: proj1-alb-sg  — inbound 0.0.0.0/0 on :80)
   ▼
[ Target Group ]  — Health check: HTTP GET "/"
   │
   ▼
[ Auto Scaling Group (2× EC2, 2 AZs) ]
   ├─ [ EC2 #1 ] (sg: proj1-ec2-sg — allows :80 only from ALB SG)
   └─ [ EC2 #2 ] (sg: proj1-ec2-sg)
</code></pre>

---

## **Implementation Details (Reviewer-Friendly)**
**Launch Template — User Data (Amazon Linux 2023):**  
<pre><code>#!/bin/bash
set -euxo pipefail
dnf -y update
dnf -y install httpd
# IMDSv2 token for metadata
TOKEN=$(curl -sS -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
INSTANCE_ID=$(curl -sS -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)
echo "&lt;h1&gt;Hi from $INSTANCE_ID&lt;/h1&gt;" &gt; /var/www/html/index.html
systemctl enable httpd
systemctl start httpd
</code></pre>

**Target Group Health Check:** **HTTP**, **Path `/`**, **Success 200**  
**ASG Health Settings:** Type = **ELB**, Grace Period ≈ **120s**

---

## **Evidence (Screenshots)**

**Security Groups**  
<img src="./screenshots/01-sg-alb-inbound.png" alt="ALB inbound 0.0.0.0/0 :80" width="900">
<img src="./screenshots/02-sg-ec2-inbound.png" alt="EC2 allows :80 from ALB SG" width="900">

**Target Group**  
<img src="./screenshots/03-tg-healthcheck.png" alt="Health check path /" width="900">
<img src="./screenshots/04-tg-healthy-targets.png" alt="Healthy targets across two AZs" width="900">

**Load Balancer**  
<img src="./screenshots/05-alb-description-dns.png" alt="ALB details and DNS name" width="900">
<img src="./screenshots/06-alb-listener.png" alt="Listener :80 forwarding to target group" width="900">

**Auto Scaling Group**  
<img src="./screenshots/07-asg-activity-replace.png" alt="ASG activity replacing a terminated instance" width="900">
<img src="./screenshots/08-asg-instances-inservice.png" alt="Two instances InService after heal" width="900">

**App Proof (refresh shows different IDs)**  
<img src="./screenshots/09-app-instanceA.png" alt="App served by instance A" width="900">
<img src="./screenshots/10-app-instanceB.png" alt="App served by instance B" width="900">

---

## **Demo (2 Minutes)**
- **Refresh** the ALB DNS; watch the **instance ID** change between two servers.  
- **Terminate** one instance (EC2 → Instances) to simulate failure.  
- Watch **Target Group → Targets**: one drains/unhealthy, the other stays healthy.  
- Watch **ASG → Activity**: it **launches a replacement**. The site stays up.  
Docs: [`docs/runbook.md`](./docs/runbook.md)

---

## **Cleanup (Avoid Charges)**
Delete in this order: **ASG → Launch Template → ALB → Target Group → Security Groups**  
Docs: [`docs/cleanup.md`](./docs/cleanup.md)

---

## **Cost Notes**
- The **ALB** has a small **hourly** cost.  
- **t2.micro** may be **Free Tier** (depending on your account).  
- For demos: capture screenshots/video, then **tear down**.

*(Tip: set up an AWS **Budget** alert in Billing → Budgets so surprises don’t happen.)*

---

## **What I Practiced / Learned**
- **Designed & deployed** a **multi-AZ**, **auto-healing** web stack on AWS (ALB + ASG) with **least-privilege** networking.  
- **Automated** server bootstrap (Amazon Linux 2023 + Apache) via **User Data**; validated auto-healing by termination and replacement.  
- Produced a **runbook & cleanup** and captured **evidence screenshots** for auditability and client hand-off.

---

## **Who This Helps**
- **Small businesses / creators** who want uptime without heavy ops.  
- **Teams** needing a minimal, auditable HA baseline before adding HTTPS, alarms, or CI/CD.  
- **Early-career engineers** who want a clear, demonstrable HA reference build.

---

## **Roadmap / V2 Upgrades**
See: **[`docs/roadmap-v2.md`](./docs/roadmap-v2.md)** (plain-English steps to add each upgrade)

---

## **Repository Structure**
<pre><code>.
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
</code></pre>
Commit + push:

bash
Copy
Edit
git add README.md
git commit -m "docs: 10/10 polish — badges, ToC, tighter sections"
git push
Step 2 — Add the plain-English Roadmap / V2 guide
bash
Copy
Edit
nano docs/roadmap-v2.md
Paste everything below, then Ctrl+O, Enter, Ctrl+X.

markdown
Copy
Edit
# Roadmap / V2 Upgrades (Elementary + Console-First)

Goal: add “real production” touches in small, safe steps — using the AWS Console.

---

## 1) HTTPS + Custom Domain (padlock + redirect)

**What it is:** Use TLS so your site is https://… with a padlock.  
**Why:** Trust + SEO + security.

**Steps (Console):**
1) **ACM (Certificates Manager)** → Request a **public certificate** for your domain (e.g., `www.example.com`).  
2) **DNS validation:** ACM shows CNAME records.  
   - If you use **Route 53**, click **Create records in Route 53**.  
   - If you use another registrar, copy the CNAME into your registrar’s DNS.  
3) Wait until ACM shows **Issued**.  
4) **EC2 → Load Balancers → proj1-alb → Listeners**  
   - **Add listener** :443 (HTTPS) → **Choose your ACM certificate** → **Default action: forward** to `proj1-tg`.  
5) **Security Group for ALB:** allow **inbound 443** from **0.0.0.0/0** (keep 80 for the redirect).  
6) **HTTP → HTTPS redirect:** Edit the **:80** listener action to **Redirect** to **HTTPS:443** (301).  
7) Test: open `https://your-domain` → padlock shows; `http://` should redirect to `https://`.

---

## 2) CloudWatch Alarms + Simple Dashboard

**What it is:** Email you when things go wrong; a one-page dashboard.  
**Why:** See problems fast.

**Steps:**
1) **SNS (Simple Notification Service)** → **Create topic** (e.g., `proj1-alerts`) → **Create subscription** with your email → click confirm in your inbox.  
2) **CloudWatch → Alarms → Create alarm**:
   - **ALB 5xx errors:** Metric = `ALB → LoadBalancer → HTTPCode_ELB_5XX_Count`. Threshold: `>= 1` for `5 minutes`. Action: notify `proj1-alerts`.  
   - **Unhealthy targets:** Metric = `TargetGroup → UnHealthyHostCount` (filter by your TG + LB). Threshold: `>= 1` for `1 minute`. Action: `proj1-alerts`.  
   - **ASG capacity:** Metric = `AutoScaling → GroupInServiceInstances` (your ASG). Threshold: `< 2` for `5 minutes`. Action: `proj1-alerts`.  
3) **CloudWatch → Dashboards → Create** (e.g., `proj1-dashboard`), add widgets for those metrics.

---

## 3) AWS WAF (basic L7 protection)

**What it is:** Web Application Firewall in front of the ALB.  
**Why:** Filter common bad traffic; optional rate limits.

**Steps:**
1) **WAF → Web ACLs → Create** (Region = your ALB’s region).  
2) Add **AWS Managed Rule groups** (Core, Common, Bad Bot). Start them in **Count** mode to observe first.  
3) Add a **rate-based rule** (e.g., “Block > 200 requests/5min per IP”).  
4) **Associate** the Web ACL with your **ALB**.  
5) After observing logs, switch from **Count** to **Block** for noisy rules.

---

## 4) Safe Rollouts (Launch Template + Instance Refresh)

**What it is:** Update the app in-place with minimal risk.  
**Why:** Roll forward confidently; easy rollback.

**Steps:**
1) **Create new Launch Template version** (v2) — e.g., change user data (page text) or package version.  
2) **ASG → Edit** to use **Latest** template version.  
3) **ASG → Instance refresh → Start**:  
   - Strategy: **Rolling**  
   - **Min healthy %**: 90 (or higher)  
   - **Instance warmup**: 120s (matches your health grace)  
4) Watch: old instances terminate gradually; new ones join healthy.  
5) Rollback: switch ASG back to previous template version and start another refresh.

---

## 5) Cost Guardrails (Budgets + Teardown)

**What it is:** Alerts when spend creeps up; checklist to turn things off.  
**Why:** Keep the bill at pennies.

**Steps:**
1) **Billing → Budgets → Create budget**  
   - Type: **Cost**; Period: **Monthly**; Amount: e.g., **$5**.  
   - Alerts: at **50% / 80% / 100%** to your email (via SNS).  
2) **Teardown checklist** (already in `docs/cleanup.md`):  
   - Delete: **ASG → Launch Template → ALB → Target Group → Security Groups**.  
   - Verify: EC2 **Instances/Volumes** and **Load balancers/Target groups** are **empty**.

---
