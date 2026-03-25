Manual Setup Step-by-Step AWS Free-Tier Project: “Hello World” Web App
Step 0: What You Need (Explained Simply)
Before starting, make sure:
You have an AWS account (sign up on Amazon Web Services)
You can log in to AWS Console
You know basic commands like:
cd (change directory)
ls (list files)
Install:
Terminal (Mac/Linux) OR
PuTTY (for Windows users)
Step 1: Create VPC (Your Private Network)
 Think of VPC as your own mini internet inside AWS
Steps:
Login to AWS Console
Search: VPC
Click → Create VPC
Fill:
Name: HelloWorldVPC
IPv4 CIDR: 10.0.0.0/16
Leave everything else default
Click Create VPC
 Done — You created your network

Step 2: Create Subnets
Subnets = smaller networks inside VPC
We create 4 subnets
🔹 Public Subnet 1
Name: PublicSubnet1
CIDR: 10.0.1.0/24
AZ: us-east-1a
🔹 Public Subnet 2
Name: PublicSubnet2
CIDR: 10.0.2.0/24
AZ: us-east-1b
🔹 Private Subnet 1
Name: PrivateSubnet1
CIDR: 10.0.3.0/24
🔹 Private Subnet 2
Name: PrivateSubnet2
CIDR: 10.0.4.0/24
Repeat process:
VPC → Subnets → Create subnet
Done — You created network layers
Step 3: Internet Gateway (IGW)
This allows internet access
Steps:
Go to Internet Gateways
Click Create
Name: HelloWorldIGW
Click Create
Attach it:
Select IGW → Actions → Attach to VPC
Choose HelloWorldVPC
 Now internet is connected
Step 4: Route Tables
🔹 Public Route Table
Create Route Table:
Name: PublicRT
VPC: HelloWorldVPC
Add Route:
Destination: 0.0.0.0/0
Target: HelloWorldIGW
Associate:
PublicSubnet1
PublicSubnet2
🔹 Private Route Table
Create: PrivateRT
No internet route
 Associate:
PrivateSubnet1
PrivateSubnet2
Done — Traffic control setup complete



Step 5: Launch EC2 Instance
This is your server
Go to EC2
Click Launch Instance
Fill details:
Name: HelloWorldServer
AMI: Amazon Linux 2023
Instance type: t2.micro (Free tier)
Network settings:
VPC: HelloWorldVPC
Subnet: PublicSubnet1
Auto-assign Public IP: ✅ Enabled
Security Group:
Create new:
SSH (22) → Your IP
HTTP (80) → Anywhere 0.0.0.0/0
Key Pair:
Create new → Download .pem
👉 Launch instance
Server is running

Step 6: Connect via SSH
Open terminal:
chmod 400 your-key.pem

ssh -i your-key.pem ec2-user@YOUR_PUBLIC_IP

You are inside the server
Step 7: Install Nginx
Run one by one:
sudo yum update -y

sudo yum install nginx -y

sudo systemctl start nginx

sudo systemctl enable nginx

Check:
sudo systemctl status nginx

Nginx running
Step 8: Create Web Page
Get system info:
INSTANCE_ID=$(curl http://169.254.169.254/latest/meta-data/instance-id)
AZ=$(curl http://169.254.169.254/latest/meta-data/placement/availability-zone)


Edit HTML:
sudo nano /usr/share/nginx/html/index.html

Paste:
<h1>Hello World</h1>
<p>Instance ID: REPLACE_WITH_INSTANCE_ID</p>
<p>AZ: REPLACE_WITH_AZ</p>

Replace values:
sudo sed -i "s/REPLACE_WITH_INSTANCE_ID/$INSTANCE_ID/" /usr/share/nginx/html/index.html

sudo sed -i "s/REPLACE_WITH_AZ/$AZ/" /usr/share/nginx/html/index.html
 Open browser:
http://YOUR_PUBLIC_IP

You will see Hello World page
Step 9: Security Check
Make sure:
Port 80 → open
Port 22 → only your IP
Step 10: Cleanup
⚠ IMPORTANT (to avoid billing)
Terminate EC2
Delete VPC
Delete IGW
Delete Subnets













PART 2 — READY DOCUMENT (FOR SUBMISSION)
You can copy this directly:
AWS Hello World Web Application Project
Objective
To deploy a simple web application using AWS Free Tier services including VPC, EC2, and Nginx.
 Architecture
VPC (10.0.0.0/16)
2 Public Subnets
2 Private Subnets
Internet Gateway
EC2 Instance (Public)
Services Used
Amazon EC2
Amazon VPC
Internet Gateway
Route Tables
Security Groups
Nginx Web Server
Implementation Steps
Step 1: Created VPC
Name: HelloWorldVPC
CIDR: 10.0.0.0/16
Step 2: Created Subnets
PublicSubnet1 → 10.0.1.0/24
PublicSubnet2 → 10.0.2.0/24
PrivateSubnet1 → 10.0.3.0/24
PrivateSubnet2 → 10.0.4.0/24
Step 3: Internet Gateway
Created and attached to VPC
Step 4: Route Tables
PublicRT → Internet access enabled
PrivateRT → No internet
Step 5: EC2 Instance
Amazon Linux 2023
t2.micro
Public subnet
Step 6: Nginx Setup
Installed and started Nginx server.
Step 7: Web Page
Created dynamic HTML page displaying:
Instance ID
Availability Zone
Security
SSH restricted to my IP
HTTP open to all

Output
Hello World webpage accessible via public IP.
Cleanup
All resources deleted to avoid charges.
Conclusion
Successfully deployed a web server using AWS Free Tier demonstrating basic cloud networking and compute services.



