---
title: "Week 2 Worklog"
date: 2026-06-08
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 objectives

* Understand how AWS Identity and Access Management (IAM) manages identities and access to AWS resources.
* Learn how to monitor costs, configure budget alerts, and estimate AWS workload costs.
* Learn the fundamental Amazon VPC components and how they connect AWS resources.
* Practice launching, configuring, and accessing an Amazon EC2 server.
* Learn how to store and protect data with Amazon S3.

### Tasks completed during the week

| Day | Tasks | Start date | Completion date | Reference material |
| --- | --- | --- | --- | --- |
| **2** | - Studied **AWS Identity and Access Management (IAM)**.<br>- Distinguished between the main IAM components:<br>&emsp;+ IAM users<br>&emsp;+ User groups<br>&emsp;+ IAM roles<br>&emsp;+ Policies<br>- Learned the principle of least privilege and the role of multi-factor authentication (MFA) in protecting AWS accounts. | 06/08/2026 | 06/08/2026 | https://000002.awsstudygroup.com/ |
| **3** | - Learned about **AWS Budgets** in Billing and Cost Management.<br>- Learned how to use the **AWS Pricing Calculator** to estimate service costs.<br>- Practiced:<br>&emsp;+ Creating a budget with cost alerts<br>&emsp;+ Creating a cost-monitoring dashboard<br>&emsp;+ Creating a cost estimate for a sample workload | 06/09/2026 | 06/09/2026 | https://000007.awsstudygroup.com/ |
| **4** | - Studied **Amazon Virtual Private Cloud (VPC)**.<br>- Explored the fundamental networking components:<br>&emsp;+ CIDR blocks<br>&emsp;+ Public and private subnets<br>&emsp;+ Route tables<br>&emsp;+ Internet gateways<br>&emsp;+ Network ACLs and security groups<br>- Sketched a basic VPC model for a web application. | 06/10/2026 | 06/10/2026 | https://000003.awsstudygroup.com/ |
| **5** | - Launched an **Amazon EC2 instance** with Amazon Linux.<br>- Created a key pair, configured a security group, and connected to EC2 through SSH.<br>- Monitored the instance state and explored the Start, Stop, Reboot, and Terminate operations.<br>- Studied **Amazon Elastic Block Store (EBS)** and its relationship with EC2.<br>- Created, attached, and inspected an EBS volume on the EC2 instance. | 06/11/2026 | 06/11/2026 | https://000004.awsstudygroup.com/ |
| **6** | - Studied the fundamental concepts of **Amazon S3**:<br>&emsp;+ Buckets and objects<br>&emsp;+ Object keys<br>&emsp;+ Storage classes<br>&emsp;+ Versioning<br>&emsp;+ Block Public Access<br>- Created a bucket and practiced uploading, downloading, and organizing objects.<br>- Reviewed access permissions and learned how to protect S3 data from unintended public access. | 06/12/2026 | 06/12/2026 | https://000057.awsstudygroup.com/ |

### Week 2 achievements

* Understood the purpose of IAM and distinguished between:
  * IAM users
  * User groups
  * IAM roles
  * Policies

* Understood the principle of least privilege and the role of MFA in protecting AWS accounts.
* Used AWS Budgets to create a budget and configure cost alerts.
* Used the AWS Pricing Calculator to create a cost estimate for a sample workload.
* Created a dashboard for monitoring AWS costs.
* Learned the fundamental Amazon VPC components:
  * CIDR blocks
  * Public subnets
  * Private subnets
  * Route tables
  * Internet gateways
  * Network ACLs
  * Security groups
* Successfully launched and configured an EC2 instance.
* Connected to an EC2 server through SSH and performed basic administration tasks.
* Understood how EBS provides block storage for EC2 and attached an EBS volume to an instance.
* Learned the Amazon S3 concepts of buckets, objects, storage classes, versioning, and Block Public Access.
* Created an S3 bucket and performed basic object upload, download, and management operations.
* Built a foundation in identity, networking, compute, and storage for studying AWS system architectures in the following weeks.
