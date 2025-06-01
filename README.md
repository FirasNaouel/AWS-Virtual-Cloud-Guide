# AWS Virtual Machine Deployment: A Crash Course

## Overview
Welcome to this guided crash course on deploying virtual machines using AWS. This is not your typical tutorial. Instead, it's designed to teach you the why and how of cloud architecture as we build it step by step. By the end, you won't just know what buttons to press—you'll understand what’s happening behind the scenes, and why each decision matters.

## Prerequisites
Before diving in, make sure you:
- Understand basic networking (IP addressing, subnets)
- Have SSH access via terminal
- Possess an AWS account (free tier is sufficient)
  
__Pro Tip:__ Choose a region closest to you for better performance. The AWS region selector is in the top-right corner of the console.

## Course Structure
We'll walk through building a basic AWS infrastructure composed of:
- A custom VPC
- Internet Gateway
- Public and private subnets
- Route tables
- EC2 instances (one public, one private)

Let’s get started!

## 1. Build Your Virtual Private Cloud (VPC)
Think of a VPC as your private corner of the AWS universe. It's like setting up a private data center with customizable networking.

### Why it matters

A VPC allows you to define your own network rules, ranges, and how resources talk to each other.
### Steps

1. Use the search bar to look up VPC.
2. Navigate to Your VPCs > Create VPC.
3. Choose VPC Only.
4. Name your VPC.
5. Choose a CIDR block between /16 and /24, we will use 12.0.0.0/16
6. Click Create VPC.

You’ve now created your virtual network.

## 2. Connect to the Internet with an Internet Gateway (IGW)
By default, your VPC is isolated from the internet. The Internet Gateway is your VPC’s bridge to the outside world.

### Why it matters
You control which parts of your network are accessible externally by routing traffic through the IGW.

### Steps

1. Go to Internet Gateways > Create Internet Gateway.
2. Name it.
3. Click Create Internet Gateway.
4. Select your new Internet Gateway.
5. Click Actions > Attach to VPC, and select your VPC.

Your VPC is now capable of internet communication—but nothing is using it yet.

## 3. Divide the Network with Subnets
Subnets break your VPC into manageable chunks. Each subnet lives in a specific availability zone.

### Why it matters
You separate responsibilities: public subnets for internet-facing services, private for internal systems.

### Steps
This guide creates two subnets for different use cases, if you only want one follow the steps for the public subnet.

1. Go to Subnets > Create Subnet.
2. Name your Subnet
3. Choose your VPC.
4. Choose an Availability Zone (e.g., ca-central-1a, any one will work).
5. Create two subnets:
   - Public Subnet (e.g., 12.0.1.0/24)
   - Private Subnet (e.g., 12.0.2.0/24)

A /24 CIDR gives you 256 addresses—plenty for a small setup. The higher the CIDR, the less addresses you will have.

## 4. Control Routing with Route Tables
Route tables dictate where your network traffic goes. Each subnet needs to be linked to one.

### Why it matters
Only subnets with a route to the IGW can talk to the internet. Others stay private by default.

### Steps
1. Go to __Route Tables__ > __Create Route Table__, and select your VPC.
2. Create two tables: one public, one private.

### Configure Public Route Table
1. Select the public route table > __Routes Tab__ > __Edit Routes__.
2. Add a route:
   - Destination: 0.0.0.0/0
   - Target > __Internet Gateway__ > __Your Internet Gateway__

### Associate Subnets
1. For each route table, select it, and go to the __Subnet Associations__ tab.
2. Click __Edit Subnet Association__.
3. Attach:
   - Public subnet to public route table
   - Private subnet to private route table


Your network now has properly segmented traffic flow.

## 5. Deploy EC2 Instances
Let’s create two virtual machines (EC2 instances): one publicly accessible, and one hidden inside your private subnet.

### Why it matters
You’ll see the contrast between public and private infrastructure firsthand. If you only want to create one, follow the steps for the public virtual machine.

### Steps
1. Use the search bar to look up __EC2__.
2. Go to __Instances__ > __Launch Instance__.
3. Name your instance.
4. Choose a free-tier-eligible AMI (e.g., Ubuntu) and keep track of the username.
5. Choose a free-tier eligible Instance Type (e.g t2.micro).
6. Create a Key Pair:
   - Name your __Key Pair__
   - Choose __RSA__
   - Download and store the __.pem__ file securely


### Networking Settings
1. Click on __Edit__.
2. Select your VPC.
- For __public instance__:
   - Select the public subnet
   - Enable Auto-assign Public IP
- For __private instance__:
   - Select the private subnet
   - Disable public IP


### Security Groups
Create a security group to define access:
- Public instance:
   - Allow SSH (port 22) from Anywhere (easier access in case of IP change)
- Private instance:
   - Allow SSH only from Custom > Public Security Group (last on the list)
     
Launch both instances.


## 6. Connect to the Public Instance
Use SSH to connect to your public VM.

Steps
1. Get the public IP of your instance.
2. In your terminal:
```bash
sudo chmod 400 /path/to/local/public-key.pem
ssh -i /path/to/local/public-key.pem ec2-user@your-public-ip
```

__Note__: Replace ec2-user with your AMI’s default user (e.g., ubuntu).


## 7. Connect to the Private Instance via Jump Host
You can’t access a private instance directly. Instead, use the public EC2 as a jump host.

### Why it matters
Jump hosts let you securely connect to private infrastructure without exposing it to the internet.

### Steps
1. Copy the private instance’s .pem key to the public instance:
```bash
sudo scp -i /path/to/local/public-key.pem /path/to/local/private-key.pem ec2-user@your-public-ip:/home/ec2-user/
```
2. SSH into the public instance:
```bash
ssh -i /path/to/local/public-key.pem ec2-user@your-public-ip
```
3. From the public EC2, SSH into the private EC2:
```bash
sudo chmod 400 /path/to/local/private-key.pem
ssh -i /path/to/local/private-key.pem ec2-user@your-private-ip
```

## Conclusion
You've now built a functioning AWS environment with a proper networking layout. You:
- Designed a VPC
- Configured public and private subnets
- Set up an Internet Gateway and route tables
- Deployed EC2 instances with controlled access

More importantly, you learned why each of these steps matters.

Cloud architecture isn’t about clicking buttons—it’s about intentional design, security, and scalability. By completing this course, you’ve taken a big step toward becoming an infrastructure-savvy cloud engineer.

The possibilities are now endless. One way to enjoy your instances is to host your own website on them. You can do so by following [this guide](https://github.com/Nsh-GaMeS/locally-hosted-website) by Nsh-GaMes.

