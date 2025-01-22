# WordPress Hosting on AWS - DevOps Project
## Overview

This project demonstrates how to host a **WordPress website on AWS**, utilizing various AWS services and best practices to ensure high availability, scalability, and security. The solution was built using services such as **EC2**, **RDS**, **EFS**, **ALB**, **Auto Scaling**, **Route 53**, and **SNS**. The deployment ensures fault tolerance and resilience by leveraging multiple availability zones.

### Key Components:
1. **Amazon VPC** with Public and Private Subnets
2. **Internet Gateway** for internet connectivity
3. **Security Groups** for secure network traffic control
4. **EC2 Instances** for hosting the WordPress application
5. **Application Load Balancer** for distributing traffic across EC2 instances
6. **Auto Scaling Group** for managing the EC2 instances
7. **EFS** for shared file storage
8. **RDS** for WordPress database
9. **Route 53** for domain management
10. **SNS** for notifications and monitoring

## Architecture Diagram

The architecture diagram provides a detailed view of the resources deployed in AWS. You can find the diagram and additional project resources in the [GitHub Repository](https://github.com/your-repository-link).

## Steps for Deployment

### 1. **VPC Setup**
   - Create a **Virtual Private Cloud (VPC)** with both **public** and **private subnets** across two availability zones.
   - Deploy an **Internet Gateway** to facilitate internet access for resources in the public subnet.
   - Configure **Security Groups** to control inbound and outbound traffic.

### 2. **Web Server Configuration (EC2 Instances)**
   - Launch EC2 instances within the **private subnets** for enhanced security.
   - Install Apache, PHP, MySQL, and WordPress as outlined in the [WordPress Installation Script](#WordPress-Installation-Script).

### 3. **NAT Gateway and ALB Setup**
   - Deploy a **NAT Gateway** in the public subnet to enable internet access for private instances.
   - Use an **Application Load Balancer** to distribute web traffic across EC2 instances in multiple availability zones.

### 4. **Database Setup (RDS)**
   - Deploy an **RDS** instance for the WordPress MySQL database in the private subnet.
   - Update the `wp-config.php` file to connect WordPress to the RDS instance.

### 5. **Auto Scaling Configuration**
   - Configure an **Auto Scaling Group** to automatically scale the EC2 instances based on demand.

### 6. **File Storage (EFS)**
   - Mount an **EFS (Elastic File System)** to the EC2 instances for storing WordPress files.

### 7. **DNS Setup (Route 53)**
   - Register a domain name and set up DNS records using **Route 53** for routing traffic to the Application Load Balancer.

### 8. **SSL Setup**
   - Secure communications with **AWS Certificate Manager (ACM)** by configuring an SSL certificate.

### 9. **Monitoring and Notifications (SNS)**
   - Set up **Simple Notification Service (SNS)** to alert about events and activities in the Auto Scaling Group.

## Scripts

### WordPress Installation Script for EC2

This script installs Apache, PHP, MySQL, and WordPress on an EC2 instance.

```bash
# Switch to root user
sudo su

# Update software packages
sudo yum update -y

# Create html directory
sudo mkdir -p /var/www/html

# Mount EFS
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install Apache
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP and necessary extensions for WordPress
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL Server
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf install -y mysql-community-server

# Start MySQL Server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Configure permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html

# Download and extract WordPress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Copy wp-config-sample.php to wp-config.php and edit the file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
sudo vi /var/www/html/wp-config.php

# Restart Apache
sudo service httpd restart
```

### Auto Scaling Group Launch Template Script

The script below installs necessary software for EC2 instances in an Auto Scaling Group:

```bash
#!/bin/bash
# Update software packages
sudo yum update -y

# Install Apache
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf install -y mysql-community-server

# Start MySQL
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Mount EFS
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Set permissions
chown apache:apache -R /var/www/html

# Restart Apache
sudo service httpd restart
```

## Setting Up Auto Scaling Group

1. Create a **Launch Template** using the script provided.
2. Set the desired EC2 instance configuration (AMI, instance type, etc.).
3. Create an **Auto Scaling Group** and associate it with the Launch Template.
4. Define scaling policies to automatically scale the number of EC2 instances based on traffic demand.

## Domain and DNS Setup

1. Register a domain name using **Route 53**.
2. Create an A record to point to the Application Load Balancer (ALB).

## Monitoring with SNS

1. Set up an SNS topic for Auto Scaling Group notifications.
2. Configure CloudWatch Alarms to trigger notifications for scaling events.

## Conclusion

This project provides a highly available and scalable WordPress deployment on AWS, incorporating best practices for security, scalability, and fault tolerance. By utilizing services like EC2, RDS, EFS, and Auto Scaling, the solution ensures that the website remains available and responsive even during traffic spikes.

For more details, please refer to the [GitHub Repository](https://github.com/your-repository-link).
