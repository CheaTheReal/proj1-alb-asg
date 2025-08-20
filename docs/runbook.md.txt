# Runbook (2-minute demo)

1) Open the ALB DNS in a browser; refresh to see instance IDs flip.
2) Terminate one instance (EC2 → Instances) to simulate failure.
3) Watch Target Group → Targets go to one healthy, one draining; ASG launches a replacement.
4) Keep refreshing the ALB page — app stays up (self-healing).
