![Alt text](/https://github.com/logunlaja26/host-static-website-aws/blob/main/Host_a_Static_Website_on_AWS.png)

---

# Host a Static Website on AWS

This repository contains a reference architecture and deployment scripts for hosting a static HTML website on AWS. The project utilizes a variety of AWS services to ensure high availability, scalability, fault tolerance, and security.

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [AWS Components](#aws-components)
- [Infrastructure Diagram](#infrastructure-diagram)
- [Prerequisites](#prerequisites)
- [Deployment Steps](#deployment-steps)
  - [1. Clone the Repository](#1-clone-the-repository)
  - [2. Review the AWS Cloud Infrastructure Setup](#2-review-the-aws-cloud-infrastructure-setup)
  - [3. Provision the AWS Resources](#3-provision-the-aws-resources)
  - [4. Run the Installation Script on EC2](#4-run-the-installation-script-on-ec2)
- [Script Details](#script-details)
- [Domain and DNS Configuration](#domain-and-dns-configuration)
- [Scaling and Monitoring](#scaling-and-monitoring)
- [Security](#security)
- [License](#license)

---

## Architecture Overview

This solution leverages various AWS services to host a static website with high availability, security, and scalability:

1. **VPC (Virtual Private Cloud)** with both **public and private subnets** across two Availability Zones (AZs).  
2. **Internet Gateway** for outbound internet access from the VPC.  
3. **Security Groups** functioning as firewalls for the instances.  
4. Use of two **Availability Zones** for fault tolerance.  
5. **Public subnets** hosting infrastructure components such as **NAT Gateway** and **Application Load Balancer (ALB)**.  
6. **EC2 Instance Connect Endpoint** for secure connections to both public and private subnets.  
7. **Private subnets** containing web servers (EC2 instances) for enhanced security.  
8. **NAT Gateway** enabling private subnets to access the internet for updates and other outbound connections.  
9. **EC2 instances** running Apache to serve the static website.  
10. **Application Load Balancer (ALB)** distributing traffic to **Auto Scaling Groups** across multiple AZs.  
11. **Auto Scaling Group** to automatically scale in/out EC2 instances based on demand.  
12. **GitHub** for source control of website files.  
13. **AWS Certificate Manager** (ACM) for SSL/TLS certificates.  
14. **Amazon Simple Notification Service (SNS)** for notifications related to the Auto Scaling Group.  
15. **Amazon Route 53** for domain registration and DNS management.

---

## AWS Components

The following AWS resources are used in this project:

- **VPC** and subnets (public, private)  
- **Internet Gateway** and **NAT Gateway**  
- **Security Groups**  
- **EC2 Instance Connect Endpoint**  
- **EC2 instances**  
- **Application Load Balancer**  
- **Target Group**  
- **Auto Scaling Group**  
- **AWS Certificate Manager**  
- **AWS SNS**  
- **Amazon Route 53**

---

## Infrastructure Diagram

A high-level architecture diagram is included in this repository (`architecture-diagram.png` or similar file). It illustrates how the various AWS components are interconnected to provide a secure and highly available hosting environment.

```
[Client] 
   |
   |  HTTPS
   v
[Route 53] --> [ALB] --> [Auto Scaling Group of EC2 Instances in Private Subnets]
   |                \
   |                 \--> [NAT Gateway -> Internet Gateway -> Internet]
   |
[SNS for Notifications]
```

---

## Prerequisites

1. **AWS Account** – You need an active AWS account with appropriate permissions (IAM user or role).  
2. **Domain Name** – Optional, but recommended. You can purchase or migrate a domain name to **Amazon Route 53**.  
3. **Git** – Required to clone the repository.  
4. **Bash terminal / SSH client** – For connecting to EC2 instances and running scripts.  
5. **Key Pair** – (Optional if using AWS EC2 Instance Connect) A key pair to SSH into the EC2 instance if needed.

---

## Deployment Steps

### 1. Clone the Repository

Clone this repository to your local machine:

```bash
git clone https://github.com/logunlaja26/host-static-website-aws.git
cd host-a-static-website-on-aws
```

### 2. Review the AWS Cloud Infrastructure Setup

Check the CloudFormation templates, Terraform files, or manual instructions (if provided) within this repo for creating the following resources in your AWS environment:

- **VPC** with subnets (public and private) in two AZs  
- **Internet Gateway** attached to the VPC  
- **NAT Gateway** placed in a public subnet  
- **Application Load Balancer** in public subnets  
- **EC2 Instance Connect Endpoint** if required for your private subnets  
- **Security Groups** for the ALB, EC2, and NAT Gateway  
- **Auto Scaling Group** referencing the ALB Target Group  

### 3. Provision the AWS Resources

Depending on how you want to set up your infrastructure, do one of the following:

- Use **AWS Console** to manually create the resources.  
- Use **AWS CLI**, **CloudFormation**, or **Terraform** scripts included in this repository to deploy resources automatically.  

Make sure you create (or have) the following:

- **VPC, Subnets, and Internet Gateway**  
- **NAT Gateway** in the public subnet  
- **Security Groups** for the ALB and EC2  
- **Load Balancer** (ALB) configured to forward traffic to a **Target Group**  
- **Auto Scaling Group** for your EC2 instances  

### 4. Run the Installation Script on EC2

Once your EC2 instances are created (either manually or via an Auto Scaling Group launch configuration/launch template), use the provided `install.sh` script (shown below) to install Apache and clone your website files onto each server.

1. **SSH** into your EC2 instance or use **EC2 Instance Connect**.  
2. Make the script executable and run it:

   ```bash
   chmod +x install.sh
   sudo ./install.sh
   ```

---

## Script Details

Below is the content of the `install.sh` (or similarly named) script included in this repository:

```bash
#!/bin/bash
# Switch to the root user to gain full administrative privileges
sudo su

# Update all installed packages to their latest versions
yum update -y

# Install Apache HTTP Server
yum install -y httpd

# Change the current working directory to the Apache web root
cd /var/www/html

# Install Git
yum install git -y

# Clone the project GitHub repository to the current directory
git clone https://github.com/logunlaja26/host-static-website-aws.git

# Copy all files, including hidden ones, from the cloned repository to the Apache web root
cp -R host-a-static-website-on-aws/. /var/www/html/

# Remove the cloned repository directory to clean up unnecessary files
rm -rf host-a-static-website-on-aws

# Enable the Apache HTTP Server to start automatically at system boot
systemctl enable httpd

# Start the Apache HTTP Server to serve web content
systemctl start httpd
```

> **Note:**  
> - Make sure the script is in the correct path (e.g., `/home/ec2-user/`) before running.  
> - You may need to update the repository URL, domain references, or any local paths based on your configuration.

---

## Domain and DNS Configuration

1. **Register a Domain** using **Amazon Route 53** or another registrar.  
2. Create a **Hosted Zone** in Route 53 for your domain.  
3. Add an **A record** (or CNAME if appropriate) pointing to the **Application Load Balancer** DNS name (e.g., `xxxxxxxx.elb.amazonaws.com`).  
4. If using **AWS Certificate Manager**, request an SSL/TLS certificate for your domain and associate it with your ALB.

---

## Scaling and Monitoring

- The **Auto Scaling Group (ASG)** ensures that the website remains available and scales based on traffic.  
- **SNS** notifications can be configured to alert you about ASG events, such as instance launch or termination.  
- **Amazon CloudWatch** can be used to monitor EC2 metrics (CPU, network, etc.) and can trigger scaling policies within the ASG.

---

## Security

- **Security Groups** restrict inbound and outbound traffic to the necessary ports (e.g., HTTP/HTTPS for your ALB, SSH if needed).  
- **Private subnets** protect your EC2 instances from direct internet exposure.  
- **EC2 Instance Connect Endpoint** provides a secure way to connect to private EC2 instances without exposing SSH ports to the internet.  
- **NAT Gateway** in a public subnet allows outbound internet access (e.g., for OS updates) from private subnets without inbound exposure.  
- **AWS Certificate Manager** secures communications with HTTPS/SSL certificates.

---

**Thank you for using this guide!**  
If you have any issues or suggestions, feel free to open an issue or create a pull request in this repository. Happy deploying!
