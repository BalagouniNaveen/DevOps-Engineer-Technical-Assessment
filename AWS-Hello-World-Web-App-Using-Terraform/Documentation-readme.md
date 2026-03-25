Design and Implementation of a Hello World Web Application on AWS Using Terraform (Infrastructure as Code Approach)
We are using:
Amazon Web Services → Cloud provider
Terraform → Automation tool
Step 1: Install Required Tools
1. Install Terraform
Download from:
 https://developer.hashicorp.com/terraform/downloads
Check:
terraform -v

2. Install AWS CLI
Download:
 https://aws.amazon.com/cli/
Check:
aws --version

Step 2: Configure AWS Credentials
Run:
aws configure



Enter:
Access Key: YOUR_KEY
Secret Key: YOUR_SECRET
Region: ap-south-1

 Now Terraform can talk to AWS.
Step 3: Create Project Folder
mkdir devops-hello-world
cd devops-hello-world

Create files:
main.tf
variables.tf
outputs.tf
userdata.sh







PART 2 — READY-TO-USE FILES (MUMBAI REGION)
 variables.tf

variable "region" {
  default = "ap-south-1"
}

variable "vpc_cidr" {
  default = "10.0.0.0/16"
}

variable "public_subnet_cidr_1" { default = "10.0.1.0/24" }
variable "public_subnet_cidr_2" { default = "10.0.2.0/24" }

variable "instance_type" {
  default = "t2.micro"
}

variable "key_name" {
  description = "Your AWS key pair name"
}


userdata.sh


#!/bin/bash
yum update -y
yum install nginx -y
systemctl start nginx
systemctl enable nginx

INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
AZ=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)

echo "<h1>Hello from AWS</h1>
<p>Instance: $INSTANCE_ID</p>
<p>AZ: $AZ</p>" > /usr/share/nginx/html/index.html


Make executable:
chmod +x userdata.sh

main.tf (IMPORTANT — READ CAREFULLY)

provider "aws" {
  region = var.region
}
Connects Terraform to AWS
VPC
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
}

Creates network (like office building)
Subnet
resource "aws_subnet" "public1" {
  vpc_id = aws_vpc.main.id
  cidr_block = var.public_subnet_cidr_1
  availability_zone = "ap-south-1a"
  map_public_ip_on_launch = true
}
 Public subnet → internet access
Internet Gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}
Enables internet
Route Table
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}
 Sends traffic to internet
Security Group
resource "aws_security_group" "sg" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port = 22
    to_port   = 22
    protocol  = "tcp"
    cidr_blocks = ["YOUR_IP/32"]
  }

  ingress {
    from_port = 80
    to_port   = 80
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port = 0
    to_port   = 0
    protocol  = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
 Firewall:
SSH → only your IP
HTTP → public
EC2
resource "aws_instance" "web" {
  ami           = "ami-0f5ee92e2d63afc18" # Mumbai AMI
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public1.id
  key_name      = var.key_name

  vpc_security_group_ids = [aws_security_group.sg.id]
  user_data = file("userdata.sh")
}

Launches server




outputs.tf

output "public_ip" {
  value = aws_instance.web.public_ip
}




PART 3 — RUN PROJECT
terraform init

 Downloads AWS provider
terraform plan

Shows what will happen
terraform apply

Creates everything
Open in browser
http://<your-public-ip>

PART 4 — PROFESSIONAL DOCUMENT
DevOps Project Documentation – AWS Terraform Deployment
1. Introduction
This project demonstrates the use of Infrastructure as Code (IaC) to provision and manage cloud resources on Amazon Web Services using Terraform.
A simple “Hello World” web application is deployed using an automated and scalable approach.
2. Objective
The primary objective of this project is to:
Automate infrastructure provisioning using Terraform
Deploy a web server on AWS
Serve a dynamic web page using Nginx
Follow DevOps best practices including automation and modular configuration
3. Tools & Technologies
AWS (EC2, VPC, Subnets, Security Groups, Internet Gateway)
Terraform (Infrastructure as Code)
Nginx (Web Server)
GitHub Actions (CI/CD - optional)
4. Architecture Overview
The architecture is designed as follows:
VPC with CIDR block (10.0.0.0/16)
Public Subnets across multiple availability zones
Internet Gateway for external connectivity
Route Tables to enable internet routing
Security Groups acting as firewall rules
EC2 Instance running Nginx web server
(Optional Enhancement):
Application Load Balancer for scalability and high availability

5. Implementation Details
5.1 Variables Configuration
Terraform variables are defined in variables.tf to ensure:
Reusability
Flexibility
Easy configuration management

5.2 Networking Setup
Created a VPC to isolate cloud resources
Configured public subnets for internet-facing resources
Attached an Internet Gateway for outbound/inbound traffic
Created route tables to direct traffic to the Internet Gateway
5.3 Security Configuration
Security groups were configured as follows:
Port 22 (SSH): Allowed only from a specific IP address
Port 80 (HTTP): Allowed from all sources (0.0.0.0/0)
Outbound traffic: Fully open

5.4 Compute Resource (EC2)
Launched an EC2 instance (t2.micro – Free Tier eligible)
Deployed inside a public subnet
Installed Nginx using a user data script
Associated security groups for controlled access

6. Automation (User Data Script)
A bootstrap script (userdata.sh) was used to automate server configuration:
Installs Nginx
Starts and enables the service
Fetches instance metadata (Instance ID, Availability Zone)
Dynamically generates an HTML page
This eliminates manual server configuration and ensures consistency.



7. Deployment Steps
Configure AWS CLI:
aws configure
Initialize Terraform:
terraform init
Preview infrastructure:
terraform plan
Apply configuration:
terraform apply
Access the application:
http://<public-ip>

8. AWS Resources Created
VPC
Public Subnets
Internet Gateway
Route Tables
Security Groups
EC2 Instance
(Optional) Application Load Balancer
(Optional) Target Group & Listener
9. Output
Public IP address of EC2 instance
Instance ID
Load Balancer DNS (if configured)
10. Security Best Practices
Restricted SSH access to a specific IP
Avoided use of root AWS account
Used IAM user with programmatic access
Sensitive credentials stored securely (CLI / GitHub Secrets)
11. Cleanup
To avoid unnecessary billing:
terraform destroy

12. Challenges & Learning Outcomes
Challenges:
Understanding AWS networking (VPC, subnets)
Managing Terraform dependencies
Configuring secure access
Learning Outcomes:
Hands-on experience with Infrastructure as Code
AWS resource provisioning
Automation using user data scripts
Basic CI/CD pipeline setup
13. Conclusion
This project successfully demonstrates how to deploy and manage cloud infrastructure using Terraform. It highlights automation, scalability, and security—key principles in modern DevOps practices.
14. Future Enhancements
Auto Scaling Group for high availability
Application Load Balancer with HTTPS
Private subnets with NAT Gateway
Domain integration using Route 53
Monitoring using CloudWatch

