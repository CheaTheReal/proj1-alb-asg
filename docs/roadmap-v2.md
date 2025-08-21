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

