# aws-elb
Project Description

This project demonstrates a highly available and automatically scalable web application deployment on AWS. A custom-configured EC2 instance was created, services were installed, and the instance was captured as an AMI. A Launch Template was built from this AMI and used by an Auto Scaling Group (ASG) to provision identical instances on demand. An internet-facing Application Load Balancer (ALB) was deployed to distribute HTTP traffic across instances, and its DNS was mapped to a custom domain for public access.

Dynamic scaling was implemented using a Target Tracking scaling policy based on 50% average CPU utilization. When CPU usage increased, the ASG automatically launched additional instances, and when load decreased, it terminated extra instances to return to the desired capacity. Stress testing verified that the application scaled out during high load and scaled in once the load normalized, ensuring cost-efficient performance and high availability.
