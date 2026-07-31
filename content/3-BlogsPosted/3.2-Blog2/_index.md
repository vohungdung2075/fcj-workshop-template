---
title: "Blog 2"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# [AWS Architecture] System Architecture Analysis of AI Learning Platform (LearnSphere)

A detailed system architecture analysis of the backend and frontend systems for **LearnSphere (AI Learning Platform)**. This architecture combines Serverless Static Hosting, Containerized Backend on EC2, automated CI/CD pipelines, and integrated AI/Database managed services.

---

### 1. System Architecture Overview

The system is deployed on AWS Cloud in the Singapore Region (`ap-southeast-1`), ensuring low latency for Southeast Asian users. The architecture consists of 4 main pillars:

- **Frontend Hosting**: Amazon S3 + Amazon CloudFront (CDN).
- **Backend Services**: Amazon EC2 (inside VPC / Public Subnet) + Amazon ECR.
- **CI/CD Pipeline & Monitoring**: GitHub Actions + Amazon CloudWatch.
- **External Services Integration**: OpenAI API + MongoDB Atlas + S3 Media Bucket.

![LearnSphere AWS System Architecture Diagram](/images/LEARNSHPHERE.png)

---

### 2. Component Analysis & Operational Workflows

#### A. Frontend Layer
- **Storage**: Static frontend assets (React / Vite / TypeScript) are hosted in an S3 Bucket: `learnsphere-fe-static`.
- **Content Delivery Network (CDN)**: Amazon CloudFront sits in front of the S3 bucket.
- **Access Flow**: When users visit the site, requests pass through CloudFront to retrieve static content cached at Edge Locations, optimizing load speeds and reducing S3 egress costs.

#### B. Backend Layer
- **VPC & Subnet**: A VPC configured with Availability Zone `ap-southeast-1a` and a Public Subnet.
- **Compute**: Node.js/Express.js backend runs on an Amazon EC2 Instance inside the Public Subnet, connected to the internet via an Internet Gateway.
- **Container Registry**: Backend Docker images are centrally stored on **Amazon ECR**. Upon deployment, the EC2 instance pulls updated container images from ECR.

#### C. CI/CD & Monitoring
- **GitHub Actions**: Serves as the CI/CD orchestration hub:
  - Automatically builds & pushes Docker Images to ECR on code pushes.
  - Triggers automated deployment to the EC2 Instance.
  - Syncs static assets to S3 buckets.
- **Amazon CloudWatch**: Collects application logs and system metrics for monitoring, alerting, and error tracing.

#### D. Data & External Integration
The EC2 Backend handles core application logic and interacts directly with auxiliary services:
- **Storage Bucket (`ai-learning-platform-vhd`)**: Dedicated S3 Bucket for storing course media (videos, PDFs, Docx) and AI platform files.
- **OpenAI API**: Backend sends API requests to OpenAI for intelligent features (AI Tutor, quiz generation, course content analysis).
- **MongoDB Atlas**: Fully managed NoSQL database hosted externally, connected directly from the EC2 backend for user and course data persistence.

---

### 3. Key Benefits & Strengths

- **Decoupled Architecture**: Serving the frontend through CloudFront/S3 delivers fast page loads independent of backend server health.
- **Automated CI/CD Pipeline**: GitHub Actions + ECR containerization guarantees consistent Dev/Staging/Prod environments and 100% automated deployment.
- **Flexible AI & Managed Integration**: Harnessing OpenAI for intelligent AI features and MongoDB Atlas for managed data persistence avoids database cluster maintenance overhead on EC2.

---

### 4. Future Architecture Optimizations

Planned enhancements for future iterations:
- **Enhanced Security**: Moving EC2 Instances into Private Subnets behind an Application Load Balancer (ALB) in the Public Subnet.
- **Scalability**: Replacing single EC2 instances with Auto Scaling Groups or AWS ECS/EKS as user traffic scales up.
- **Database Caching**: Integrating Amazon ElastiCache (Redis) to cache OpenAI and MongoDB responses, lowering API costs and response latency.

---

### REFERENCE LINKS & ORIGINAL POST

- **AWS Study Group Facebook Post**:  
  [https://www.facebook.com/groups/awsstudygroupfcj/permalink/2222081271890166/](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2222081271890166/)

![Original Post Screenshot on AWS Study Group](/images/blog2-facebook-post.png)
