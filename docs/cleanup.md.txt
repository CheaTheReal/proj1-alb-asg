# Cleanup (avoid charges)

Delete in this order:
1) Auto Scaling Group: proj1-asg (wait for instances to terminate)
2) Launch Template: proj1-lt
3) Load Balancer: proj1-alb (wait until gone)
4) Target Group: proj1-tg
5) Security Groups: proj1-ec2-sg, proj1-alb-sg

Quick check: EC2 → Instances = none; Load balancers/Target groups = none.
