---
title: "Blog 3"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# [AWS Security & Network] Network Segmentation & Infrastructure Security Analysis for AI Learning Platform

An in-depth analysis of the **Network Architecture** and **Infrastructure Security Strategy** on AWS for the **LearnSphere** project. A system serving static frontend users, processing backend application logic on EC2, and communicating with external APIs (OpenAI, MongoDB Atlas) requires crystal-clear security boundaries.

![LearnSphere AWS Network Architecture & Security Segmentation Diagram](/images/LEARNSHPHERE.png)

---

### 1. Network Design (VPC & Subnets)

The infrastructure is contained within a Virtual Private Cloud (VPC) located in the Singapore Region (`ap-southeast-1`):

- **Public Subnet & Internet Gateway**: The EC2 Application Server resides inside a Public Subnet in Availability Zone `ap-southeast-1a`. An Internet Gateway (IGW) enables the EC2 instance to receive inbound HTTP/HTTPS traffic and initiate outbound connections to the internet.
- **Latency Optimization**: Deploying the entire infrastructure in the Singapore region minimizes latency for end-users in Vietnam and Southeast Asia.

---

### 2. Storage Isolation Strategy (S3 Buckets Isolation)

A key highlight in our design is isolating assets into two distinct S3 Buckets rather than sharing a single bucket:

- **Frontend Bucket (`learnsphere-fe-static`)**: Houses compiled static assets (HTML, CSS, JS, Bundle Assets). The bucket is not publicly exposed; instead, content is served exclusively via **Amazon CloudFront**, hiding origin S3 buckets and mitigating direct resource scanning attacks.
- **AI Data Bucket (`ai-learning-platform-vhd`)**: Stores course media files (videos, PDFs, Docx), VHD files, and AI platform resources. Strict IAM policies restrict access exclusively to the EC2 Backend Instance (Read/Write), blocking unauthorized external access.

---

### 3. Outbound Traffic Control & Integration Security

The EC2 Backend instance acts as a client initiating outbound requests to third-party services:

- **OpenAI API Integration**: Outbound traffic flows via the Internet Gateway to send prompts and receive completions from OpenAI.
- **MongoDB Atlas Integration**: EC2 establishes secure out-of-AWS database connections to MongoDB Atlas.
- **Outbound Traffic Security**: Connection strings and API keys are never hardcoded in source code. Secrets are injected at runtime via **AWS Systems Manager Parameter Store**, **Secrets Manager**, or secure Environment Variables on EC2. Furthermore, IP Whitelisting is enforced on MongoDB Atlas to restrict access exclusively to the EC2 Elastic IP address.

---

### 4. Logging, Auditing & Observability

For operational safety, all EC2 metrics and logs are pushed to **Amazon CloudWatch**:

- **System & Container Logs**: Enables early detection of abnormal access patterns or application runtime errors.
- **Resource Metrics**: Monitors CPU, RAM, and Network I/O metrics to proactively detect potential Denial of Service (DDoS) attempts or resource exhaustion.

---

### 5. Security Upgrade Roadmap

In future iterations, our team plans to enhance security further:

- **Private Subnet Migration**: Placing EC2 instances into Private Subnets behind an Application Load Balancer (ALB) in the Public Subnet, utilizing NAT Gateways for outbound traffic.
- **AWS WAF Integration**: Deploying AWS WAF in front of CloudFront and ALB to block common web exploits like SQL Injection, Cross-Site Scripting (XSS), and botnet attacks.

---

### REFERENCE LINKS & ORIGINAL POST

- **AWS Study Group Facebook Post**:  
  [https://www.facebook.com/groups/awsstudygroupfcj/permalink/2225782871520006](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2225782871520006)

![Original Post Screenshot on AWS Study Group](/images/blog3-facebook-post.png)
