# Automated Apache Web Server Deployment on EC2 using S3 and AWS Session Manager

## Project Overview

This mini project demonstrates how to automate the deployment of an Apache Web Server on an Amazon EC2 instance using **EC2 User Data**, **Amazon S3**, **IAM Roles**, and **AWS Systems Manager Session Manager**.

The goal of this project is to deploy and manage a web server **without using SSH or key pairs**, while keeping configuration centralized and automated. This reflects real-world DevOps and enterprise operational practices.

## What This Project Covers

* Automated Apache installation using EC2 User Data
* Centralized Apache configuration stored in Amazon S3
* Secure EC2 access using AWS Systems Manager Session Manager
* IAM Role-based access (no credentials stored on the instance)
* Fully automated instance bootstrap on launch

## Architecture (Enterprise-Ready, Using Project Scope Only)

### Architecture Components

* **Amazon EC2** – Hosts the Apache Web Server
* **Amazon S3** – Stores Apache configuration file (`httpd.conf`)
* **IAM Role** – Grants EC2 permission to access S3 and Systems Manager (no hardcoded credentials)
* **AWS Systems Manager (Session Manager)** – Secure instance access without SSH
* **EC2 User Data** – Automates installation and configuration at launch

### Architecture Flow

1. DevOps engineer launches an EC2 instance
2. EC2 User Data script runs automatically on boot
3. Apache Web Server is installed
4. Apache configuration (`httpd.conf`) is downloaded from Amazon S3
5. Custom `DocumentRoot` is applied (e.g. `/webapplication` instead of default `/var/www/html`)
6. Apache service starts and is enabled
7. Administrators access the instance securely via Session Manager
8. End users access the web server over HTTP

## Prerequisites

* AWS account
* Basic knowledge of EC2, S3, and IAM
* AWS CLI access (for uploading files to S3)

## Step 1: Create S3 Bucket for Configuration

Create an Amazon S3 bucket to store Apache configuration files. Use **dummy / non-sensitive names** suitable for public documentation.

Example bucket name:
  ```
  example-apache-config-bucket
  ```

Upload the Apache configuration file:
  ```
  httpd.conf
  ```

Example object path:
  ```
  s3://example-apache-config-bucket/httpd.conf
  ```

### Configuration Management (Enterprise Practice)

* `httpd.conf` is treated as a **centralized configuration artifact**
* Any future Apache configuration changes are made **directly in the S3 object**
* New EC2 instances automatically pull the updated configuration on launch

## Step 2: Create IAM Role for EC2

Create an IAM role and attach it to the EC2 instance.

### Required Permissions

* AmazonSSMManagedInstanceCore
* AmazonS3ReadOnlyAccess (or scoped access to the bucket)

This allows the EC2 instance to:

* Connect to Systems Manager
* Read configuration files from S3

## Step 3: Launch EC2 Instance

### Instance Configuration

* **AMI**: Amazon Linux 2023
* **Instance Type**: t3.micro
* **Key Pair**: None (not required)
* **IAM Role**: Attach the role created earlier

### Security Group

* Allow inbound HTTP (port 80)
* No SSH access

## Step 4: EC2 User Data Script

Add the following script in the **User Data** section during EC2 launch:

  ```bash
  #!/bin/bash
  yum install -y httpd
  
  mkdir /webapplication
  
  aws s3 cp s3://example-apache-config-bucket/httpd.conf /etc/httpd/conf/httpd.conf
  
  systemctl start httpd
  systemctl enable httpd
  ```

### What This Script Does

* Installs Apache Web Server
* Creates a custom application directory (`/webapplication`)
* Downloads Apache configuration from S3
* Applies a non-default `DocumentRoot`
* Starts and enables Apache on boot

## Step 5: Access EC2 Using Session Manager

Once the instance is running:

1. Go to **EC2 Console**
2. Select the instance
3. Click **Connect**
4. Choose **Session Manager**
5. Click **Connect**

This provides secure shell access without SSH keys or port 22.

## Step 6: Verification

### Verify Apache Service

  ```bash
  systemctl status httpd
  ```

### Access Web Server

Open a browser and visit:

  ```
  http://<EC2-Public-IP>
  ```

Apache should be running successfully.

## Key DevOps Concepts Demonstrated

* Infrastructure automation using EC2 User Data
* Centralized configuration management using Amazon S3
* Secure access using AWS Systems Manager Session Manager
* IAM role-based permissions (no embedded credentials)
* Custom Apache `DocumentRoot` management
* Immutable and repeatable deployments

## Project Summary

This project demonstrates a simple but enterprise-aligned automation workflow for deploying and managing a web server on AWS. It avoids manual intervention, removes SSH access risks, and uses AWS-native services to achieve secure and automated operations.
