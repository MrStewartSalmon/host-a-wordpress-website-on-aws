Hosting a WordPress Website on AWS
This repository contains the resources and scripts used to deploy a WordPress website on Amazon Web Services (AWS). The project leverages various AWS services to ensure high availability, scalability, and security for the WordPress application.

Architecture Overview
The WordPress website is hosted on EC2 instances within a highly available and secure architecture that includes:

Virtual Private Cloud (VPC): Public and private subnets across two Availability Zones (AZs) for fault tolerance and high availability.
Internet Gateway: Allows communication between instances in the VPC and the internet.
Security Groups: Acts as a virtual firewall to control inbound and outbound traffic.
Public Subnets: Used for the NAT Gateway and Application Load Balancer, facilitating external access and load balancing.
Private Subnets: Houses the web servers, enhancing security.
EC2 Instance Connect Endpoint: Enables secure SSH access.
Application Load Balancer: Distributes incoming web traffic across multiple EC2 instances.
Auto Scaling Group: Automatically adjusts the number of EC2 instances based on traffic, ensuring scalability and resilience.
Amazon RDS: Managed relational database service.
Amazon EFS: Scalable, elastic file storage system.
AWS Certificate Manager: Manages SSL/TLS certificates.
AWS Simple Notification Service (SNS): Sends notifications related to Auto Scaling Group activities.
Amazon Route 53: Manages domain name registration and DNS.
