# AWS Auto Scaling Group with ALB — Step-by-Step Guide

## 1. Create an EC2 instance + install your app

Launch EC2 → Amazon Linux 2 / Ubuntu.

Security Group:

- Allow HTTP (80) from Anywhere or ALB SG.
- Allow SSH (22) for your access.

Connect to instance → install your app/service.

Example:

    sudo yum install httpd -y
    sudo systemctl start httpd
    sudo systemctl enable httpd
    echo "Hello from $(hostname)" > /var/www/html/index.html

---

## 2. Create an AMI from the configured EC2

Go to EC2 → Instances.

Select your configured instance → Actions → Image → Create Image (AMI).

Name it → create → wait for AMI to finish.

---

## 3. Create a Launch Template (LT)

Go to EC2 → Launch Templates → Create launch template.

Select:

- The AMI you just created.
- Instance type (t2.micro for testing).
- Security group (same ports).
- Key pair (optional).

Save.

---

## 4. Create an HTTP Application Load Balancer

Go to EC2 → Load Balancers → Create Load Balancer → Application Load Balancer.

Choose:

- Scheme: Internet-facing
- Listener: HTTP : 80
- Select 2 public subnets in different AZs.

Create a Target Group:

- Type: Instance
- Health Check: HTTP /

Attach Target Group to ALB.

---

## 5. Create the Auto Scaling Group (ASG)

EC2 → Auto Scaling Groups → Create ASG.

Select your Launch Template.

Choose VPC + public subnets.

Attach ALB Target Group.

Define desired sizes:

- Desired capacity: 1
- Minimum: 1
- Maximum: 5

Note: Scaling policy can be set during creation and after deployment as well.

---

## 6. Add Target-Tracking Scaling Policy (CPU = 50%)

Under Automatic scaling → choose:

- Policy type: Target tracking scaling
- Metric type: Average CPU utilization
- Target value: 50

Set warm-up → ~120–300 seconds.

Save.

This means:

- If CPU > 50% → scale out
- If CPU < 50% (after warmup) → scale in

AWS automatically handles both directions.

---

## 7. Map ALB DNS to Your Custom Domain (GoDaddy / any DNS)

Copy ALB DNS name (e.g., asg-alb-test-ALB-736913041.us-east-1.elb.amazonaws.com).

Go to GoDaddy DNS settings.

Add a CNAME record:

- Name: service
- Value: ALB DNS name
- TTL: 600 sec

Wait ~5 minutes for DNS propagation.

Access your site via:

    http://service.cloudcrafting.xyz
<img width="1914" height="1051" alt="image" src="https://github.com/user-attachments/assets/4d2ac21a-82d7-4d85-9ff5-af77069c9d18" />

---

## 8. Stress Test the Instance (to trigger scaling OUT)

Connect to any EC2 in the ASG:

    sudo amazon-linux-extras install epel -y
    sudo yum install stress -y
    stress --cpu 4 --timeout 300

This drives CPU above 80–100%.

<img width="482" height="308" alt="image" src="https://github.com/user-attachments/assets/05df84be-3a02-4470-9372-b1938313082f" />

What Will Happen:

- ALB health checks keep serving traffic.
- ASG sees CPU > 50% target → launches new EC2 instances.

New instances automatically:

- Register into Target Group

<img width="1827" height="468" alt="image" src="https://github.com/user-attachments/assets/57385a7f-8091-4da0-8c49-0b8d426ef3d2" />

- Start serving traffic
- Reduce average CPU load

ASG → scaled OUT. The created instances are automatically reduced when load on ec2 is reduced.

<img width="1815" height="466" alt="image" src="https://github.com/user-attachments/assets/a8eb75c2-b775-4499-8904-c5bdfa1d42bc" />

---

## 9. Stop the Stress After a While (to trigger scaling IN)

When CPU drops and stays below 50%:

- ASG waits for cooldown/warm-up period.
- ASG terminates extra instances.

<img width="1815" height="466" alt="image" src="https://github.com/user-attachments/assets/a8eb75c2-b775-4499-8904-c5bdfa1d42bc" />

System returns to Desired capacity = 1. As it can be seen in the above image that the instance count goes back to 1 as its Ideal Count which was set as 1.

<img width="1821" height="406" alt="image" src="https://github.com/user-attachments/assets/7f033a00-344d-4c58-9319-43dc248cd996" />

ASG → scaled IN.
