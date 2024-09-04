# Hosting an WordPress Website on AWS

This repository contains the resources and scripts used to deploy a WordPress website on Amazon Web Services (AWS). The project leverages various AWS services to ensure high availability, scalability, and security for the WordPress application.

## Architecture Overview

The WordPress website is hosted on EC2 instances within a highly available and secure architecture that includes:

- **Virtual Private Cloud (VPC):** Public and private subnets across two Availability Zones (AZs) for fault tolerance and high availability.
- **Internet Gateway:** Allows communication between instances in the VPC and the internet.
- **Security Groups:** Acts as a virtual firewall to control inbound and outbound traffic.
- **Public Subnets:** Used for the NAT Gateway and Application Load Balancer, facilitating external access and load balancing.
- **Private Subnets:** Houses the web servers, enhancing security.
- **EC2 Instance Connect Endpoint:** Enables secure SSH access.
- **Application Load Balancer:** Distributes incoming web traffic across multiple EC2 instances.
- **Auto Scaling Group:** Automatically adjusts the number of EC2 instances based on traffic, ensuring scalability and resilience.
- **Amazon RDS:** Managed relational database service.
- **Amazon EFS:** Scalable, elastic file storage system.
- **AWS Certificate Manager:** Manages SSL/TLS certificates.
- **AWS Simple Notification Service (SNS):** Sends notifications related to Auto Scaling Group activities.
- **Amazon Route 53:** Manages domain name registration and DNS.

## Deployment Scripts

### WordPress Installation Script

This script is used for the initial setup of the WordPress application on an EC2 instance. It installs Apache, PHP, MySQL, and mounts the Amazon EFS to the instance.

```bash
# Become root user
sudo su

# Update software packages
sudo yum update -y

# Create HTML directory
sudo mkdir -p /var/www/html

# Environment variable
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com

# Mount EFS to HTML directory
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install Apache web server, enable and start it
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP 8 and necessary extensions for WordPress
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install MySQL 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 

# Install MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html 

# Download WordPress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Create wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# Edit wp-config.php file
sudo vi /var/www/html/wp-config.php

# Restart web server
sudo service httpd restart
```

### Auto Scaling Group Launch Template Script

This script is included in the launch template for the Auto Scaling Group, ensuring that new instances are configured correctly with the necessary software and settings.

```bash
#!/bin/bash
# Update software packages
sudo yum update -y

# Install Apache web server, enable and start it
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP 8 and necessary extensions for WordPress
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install MySQL 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 

# Install MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Environment variable
EFS_DNS_NAME=fs-00a856dd3e1b922c6.efs.eu-west-1.amazonaws.com

# Mount EFS to HTML directory
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Set permissions
chown apache:apache -R /var/www/html

# Restart web server
sudo service httpd restart
```

## Step-by-Step Process

Follow these steps to deploy the WordPress website on AWS using the provided scripts and resources:

1. **Clone the Repository**
   
   ```bash
   git clone https://github.com/your-username/your-repository.git
   cd your-repository
   ```

2. **Set Up AWS Resources**
   
   - **Create a Virtual Private Cloud (VPC):** Set up a VPC with public and private subnets across two Availability Zones (AZs) for high availability.
   - **Configure Internet Gateway:** Attach an Internet Gateway to the VPC to enable internet access.
   - **Set Up Security Groups:** Define security groups to control inbound and outbound traffic for your instances.
   - **Create Subnets:**
     - **Public Subnets:** For the NAT Gateway and Application Load Balancer.
     - **Private Subnets:** For hosting the web servers.
   - **Establish EC2 Instance Connect Endpoint:** Enable secure SSH access to your EC2 instances.

3. **Deploy Amazon RDS and EFS**
   
   - **Amazon RDS:** Set up a managed relational database for WordPress.
   - **Amazon EFS:** Create an Elastic File System for scalable file storage and mount it to your EC2 instances.

4. **Configure the Application Load Balancer**
   
   - **Create an Application Load Balancer (ALB):** Distribute incoming traffic across multiple EC2 instances.
   - **Set Up Target Groups:** Define target groups for the ALB to manage the EC2 instances effectively.
   - **Attach SSL/TLS Certificates:** Use AWS Certificate Manager to manage your SSL/TLS certificates for secure connections.

5. **Set Up Auto Scaling Group**
   
   - **Create a Launch Template:** Use the provided Auto Scaling Group Launch Template Script to configure new instances.
   - **Define Scaling Policies:** Set policies to automatically adjust the number of EC2 instances based on traffic load.
   - **Configure SNS Notifications:** Set up AWS Simple Notification Service (SNS) to receive notifications related to Auto Scaling activities.

6. **Deploy WordPress on EC2 Instances**
   
   - **Initial Setup:** Use the WordPress Installation Script to set up WordPress on your primary EC2 instance.
   - **Auto Scaling Configuration:** Ensure that the launch template script is correctly configured to deploy additional instances as needed.

7. **Domain Name and DNS Management**
   
   - **Amazon Route 53:** Register your domain name and configure DNS settings to point to the Application Load Balancer.

8. **Finalize and Test Deployment**
   
   - **Access the Website:** Navigate to the Load Balancer's DNS name in your browser to access the WordPress website.
   - **Verify High Availability and Scalability:** Test the setup by simulating traffic and ensuring that the Auto Scaling Group adjusts the number of instances accordingly.

---

This **README** provides a comprehensive overview and step-by-step guide to deploying a WordPress website on AWS using the provided scripts and resources.
