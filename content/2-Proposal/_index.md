---
title: "Proposal"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# LearnSphere - Online Learning Platform Integrated with AI
## An AWS Solution for Next-Gen E-Learning & AI Integration

### 1. Executive Summary

**LearnSphere** is a next-generation AI-integrated online learning platform designed for students, tutors, and administrators. The system centralizes course management, lessons, media assets (video, PDF/DOCX documents), quizzes, student progress tracking, and interactive discussions within a modern Single Page Application (SPA). The AI Assistant empowers students to ask context-aware questions based on lesson materials, automatically summarizes documents, and assists tutors in generating quizzes by difficulty level.

The solution utilizes a High Availability (HA) architecture deployed in the AWS Singapore Region (`ap-southeast-1`). The React/Vite/TypeScript frontend is compiled into static assets, stored in a private Amazon S3 Bucket Frontend, and globally distributed via Amazon CloudFront. The official domain `https://www.learnspherev2.id.vn` is managed via TenTen DNS, CNAME-routed to CloudFront, and secured with SSL/TLS HTTPS certificates issued by AWS Certificate Manager (ACM).

The Node.js/Express backend is containerized into immutable Docker images stored in Amazon ECR, running within an Auto Scaling Group maintaining a minimum of 2 EC2 instances across 2 Availability Zones (`ap-southeast-1a` and `ap-southeast-1b`) in Private Subnets, fronted by an Internet-facing Application Load Balancer. Course media assets reside in a separate private Amazon S3 Media Bucket, securely accessed via Presigned URLs. Business data is managed in MongoDB Atlas, while Groq API powers high-speed Generative AI inference. The CI/CD pipeline is fully automated via GitHub Actions utilizing GitHub OIDC and AWS Systems Manager Parameter Store with Instance Refresh (Zero-downtime deployment & Auto Rollback).

---

### 2. Problem Statement

Traditional E-Learning platforms face major operational and technical bottlenecks:

- **Fragmented Learning Experience**: Disjointed workflows across separate video streaming sites, document shares, quiz tools, and generic AI chatbots unaligned with course content.
- **Manual Overhead**: Tutors spend significant hours reviewing documents, preparing summaries, drafting quizzes, and manually tracking student progress.
- **Inefficient Document & Media Ingestion**: Lack of automated text extraction/OCR for PDF, DOCX, and scanned documents. Streaming large video binaries directly through the backend server causes network bottlenecks and service crashes.
- **Single Point of Failure (SPOF)**: Single EC2 backend instances risk total outage during hardware failures or unexpected traffic spikes.
- **Deployment & Security Risks**: Manual deployments, long-lived access keys, or credential leaks increase downtime risks and breach vulnerability.

LearnSphere resolves these challenges through a unified multi-role platform, secure media storage, document-bounded AI, and a 100% automated Multi-AZ High Availability cloud architecture on AWS.

---

### 3. Objectives & Scope

#### 3.1. System Objectives

- Deploy a responsive SPA accessible securely at **`https://www.learnspherev2.id.vn`** with HTTPS enforcement.
- Support 3 distinct user roles (`student`, `tutor`, `admin`) with fine-grained access control.
- Upgrade backend architecture to Multi-AZ High Availability (HA) behind an Application Load Balancer and Auto Scaling Group (minimum 2 instances in Private Subnets).
- Secure storage resources with Private S3 Buckets (Frontend & Media), Origin Access Control (OAC), and Presigned Multipart Uploads.
- Integrate AI (Groq API, OpenAI API) for context-aware Q&A, document summarization, Vietnamese PDF OCR, and automated quiz generation.
- Build a 100% automated CI/CD pipeline using GitHub Actions with GitHub OIDC, zero-downtime Instance Refresh deployments, and automated rollback upon health check failure.
- Monitor infrastructure health and application logs via Amazon CloudWatch, triggering instant email alerts via Amazon SNS.

#### 3.2. Functional Scope

| Role | Key Features |
| --- | --- |
| **Student** | Course enrollment, lesson/video viewing, document reading, progress tracking, quizzes, AI Assistant interaction, course discussion. |
| **Tutor** | Course/lesson/media creation and management; student approvals; AI-powered document summary & quiz generation; detailed student quiz reports. |
| **Admin** | User account management and role assignment; course auditing; system metrics, S3 storage monitoring, CloudWatch log auditing. |
| **Platform** | JWT Authentication (HttpOnly Secure Cookie), S3 Presigned/Multipart Upload, Orphan cleanup, AI Rate limit, GitHub OIDC CI/CD, Multi-AZ ASG & ALB Health Checks. |

---

### 4. System Architecture

![LearnSphere Production High Availability Architecture Diagram](/images/LEARNSHPHERE.png)

**Figure 1. LearnSphere Production High Availability Architecture and Service Flow.**

> **Future Architecture Evolution (AWS Native Roadmap):**  
> In future iterations, our team plans to migrate **MongoDB Atlas to Amazon DynamoDB** (utilizing the `Dynamoose` ODM library connected internally via VPC Gateway Endpoints) and migrate **Groq API to Amazon Bedrock** (direct Claude 3.5 access with built-in Bedrock Guardrails & Knowledge Bases for RAG). Additionally, integrating **AWS WAF** at the edge and employing VPC Interface Endpoints will optimize security and infrastructure costs.

#### 4.1. User Request Flow

1. User browser resolves `https://www.learnspherev2.id.vn` via TenTen DNS pointing to Amazon CloudFront CDN.
2. HTTPS connections are encrypted using SSL/TLS certificates issued by AWS Certificate Manager (ACM).
3. CloudFront fetches static Frontend assets from Amazon S3 Bucket Frontend via Origin Access Control (OAC).
4. API requests matching `/api/*` bypass cache and forward directly to the Internet-facing Application Load Balancer via HTTPS port 443 (or `origin.learnspherev2.id.vn`).
5. ALB balances traffic evenly across EC2 Backend Instances running inside 2 Private Subnets across 2 Availability Zones (`ap-southeast-1a` and `ap-southeast-1b`).
6. EC2 Backend handles business logic, JWT authentication, MongoDB Atlas queries, and Groq/OpenAI API requests via 2 independent NAT Gateways per AZ.
7. For course media (video, PDF, thumbnails), the backend generates short-lived Presigned URLs allowing browsers to upload/download directly with Amazon S3 Media Bucket.

#### 4.2. CI/CD & Deployment Automation Flow

1. Developer pushes code to the `main` branch on GitHub.
2. GitHub Actions authenticates with AWS via GitHub OIDC (no static keys), assuming temporary IAM role credentials.
3. Workflow executes Unit Tests, builds Docker Images tagged by Git SHA, and pushes to Amazon ECR.
4. Updates new image tag in AWS Systems Manager (SSM) Parameter Store.
5. Triggers Auto Scaling Instance Refresh: ASG launches fresh EC2 instances using the updated Launch Template, performing `/health/ready` checks via Target Group. Once healthy, old instances terminate (launch-before-terminate). Automatically rolls back if health checks fail.
6. Builds React Frontend, syncs static build to Amazon S3 Bucket Frontend, and invalidates CloudFront Cache.

---

### 5. Technical Component Design

| Component | Service / Technology | Technical Role |
| --- | --- | --- |
| **Frontend** | React 18, TypeScript, Vite, Tailwind CSS | Client-side SPA, role-based UI, direct S3 upload, quizzes, AI Assistant. |
| **Backend** | Node.js, Express 5, Docker | REST API, JWT auth, business logic, presigned URLs, AI orchestration, `/health/ready` checks. |
| **Compute / HA** | Amazon EC2 (`t3.small`), ASG, ALB | Multi-AZ Docker container execution in Private Subnets, auto-scaling (2-4 instances), load balancing. |
| **Container Registry** | Amazon ECR | Immutable Docker Image storage tagged by Git SHA. |
| **Database** | MongoDB Atlas, Mongoose ODM | Stores User, Course, Lesson, Enrollment, Quiz, Attempt, Progress, Notification, AI Message. |
| **Storage** | 2 Private Amazon S3 Buckets | Amazon S3 Frontend (static assets) & Amazon S3 Media (videos, PDFs, DOCX, avatars). |
| **CDN & DNS** | Amazon CloudFront, TenTen DNS, ACM | Global CDN caching, domain management `www.learnspherev2.id.vn`, SSL HTTPS certificate management. |
| **Networking** | AWS VPC, 2 Public & 2 Private Subnets, 2 NAT Gateways | Secure Multi-AZ network segmentation, isolating EC2 Backend from direct internet access. |
| **Configuration** | AWS SSM Parameter Store | Centralized management of encrypted `.env` parameters and production Docker image tags. |
| **CI/CD** | GitHub Actions, AWS OIDC, Instance Refresh | Automated test, build, ECR push, zero-downtime deployment, auto-rollback. |
| **Monitoring** | Amazon CloudWatch, Amazon SNS | Log collection, CPU/ALB Health alarms, email alert notifications. |

---

### 6. Technical Implementation & Security

- **Network Segmentation**: EC2 Backend instances reside exclusively inside Private Subnets with no public IPs. Inbound access is strictly routed through ALB in Public Subnets.
- **OIDC Authentication & IAM Least Privilege**: Short-lived GitHub OIDC credentials for CI/CD. EC2 instances assume IAM Role restricted to SSM read, CloudWatch log write, and S3 media access.
- **Data & Storage Security**: S3 Buckets enable *Block Public Access* with CloudFront OAC. Media is accessible only via short-lived Presigned URLs (15-60 minutes).
- **Centralized Secrets Management**: Secrets are never hardcoded; production environment variables are stored encrypted in SSM Parameter Store and fetched at container runtime.
- **Outbound Control & Whitelisting**: Each AZ utilizes an independent NAT Gateway. MongoDB Atlas enforces IP Whitelisting, allowing access exclusively from the Elastic IPs of AWS NAT Gateways.

---

### 7. Implementation Roadmap & Milestones

| Phase | Timeline | Activities | Deliverables |
| --- | --- | --- | --- |
| **1. AWS & VPC Foundation** | Weeks 1–4 | Multi-AZ VPC, Public/Private Subnets, NAT Gateways, ECR, S3 Buckets, ACM Certificate, OIDC setup. | Cloud infrastructure ready. |
| **2. Backend & Containerization** | Week 5 | Node.js Docker containerization, `/health/ready` check, MongoDB Atlas & S3 Presigned URL integration. | Business API containerized. |
| **3. AI & Document Processing** | Week 6 | Groq API integration, PDF/DOCX text extraction, Tesseract.js OCR, AI Tutor & Quiz Generator. | Context-bounded AI functional. |
| **4. Frontend & User Experience** | Week 7 | React/Vite UI implementation, course/progress/quiz management, CloudFront CDN & S3 OAC integration. | Responsive SPA complete. |
| **5. Production HA Deployment** | Week 8 | ALB setup, Launch Template, Multi-AZ ASG, SSM Parameter Store, GitHub Actions CI/CD with Instance Refresh. | System live on `www.learnspherev2.id.vn`. |
| **6. Verification & Final Report** | Week 9 | End-to-end testing, load testing, auto-rollback verification, documentation & Workshop completion. | Final deliverables accepted. |

---

### 8. Operational Cost Analysis

To optimize expenses and ensure budget efficiency for the LearnSphere project during the workshop phase, the system operates under a controlled Multi-AZ scenario in the AWS Singapore Region (`ap-southeast-1`). The estimated total monthly expenditure is summarized below:

- **Primary Infrastructure Subtotal (Compute, Network, Storage, ALB)**: $53.42 / month (including 2 EC2 `t3.small`, 2 NAT Gateways, 1 ALB, S3 Standard, ECR, and CloudWatch Logs running 240h/month).
- **Contingency Buffer (5%)**: $2.67 / month (covering unexpected data transfer, requests, and additional logging).
- **TOTAL ESTIMATED WORKSHOP COST**: $56.09 / month (maintaining a target range of $50–$60 / month).

This financial model balances High Availability Multi-AZ infrastructure with effective cost control. The budget is primarily allocated to NAT Gateways and the Application Load Balancer to enforce network security in Private Subnets. Detailed resource breakdowns and advanced cost optimization strategies are analyzed in depth in Section 5.11. Cost Analysis.

---

### 9. Risk Assessment & Optimization

| Risk Factor | Impact | Likelihood | Mitigation Strategy & Architecture Design |
| --- | --- | --- | --- |
| **EC2 Instance Hardware Failure** | Low | Medium | Auto Scaling Group automatically replaces unhealthy instances based on ALB/EC2 Health Checks. |
| **Deployment Code Failure Outage** | Medium | Low | Instance Refresh (Launch-before-terminate) mechanism guarantees zero downtime & auto-rollback. |
| **High NAT Gateway Cost** | Medium | High | Free S3 Gateway Endpoint handles S3 traffic; future roadmap utilizes VPC Endpoints for internal AWS services. |
| **Memory Bottleneck during PDF OCR** | Medium | Medium | Limit file upload sizes, process OCR sequentially on dedicated workers, enforce timeout & memory caps. |
| **Credential / Secret Leakage** | High | Low | GitHub OIDC for CI/CD, encrypted SSM Parameter Store for secrets, zero static access keys on servers. |

---

### 10. Expected Outcomes & Operational Value

- **Secure Access & Performance**: Fully operational application on **`https://www.learnspherev2.id.vn`**, 100% HTTPS encryption, instant frontend loads via CloudFront Edge Caching.
- **High Availability Infrastructure**: Backend isolated in Private Subnets, automatically load balanced via ALB, auto-scaled across 2 AZs via ASG.
- **100% Automated CI/CD**: Source code deployment from GitHub to AWS runs seamlessly with OIDC security, zero downtime, and instant automated rollback upon error detection.
- **Context-Bounded AI Integration**: AI Assistant provides accurate answers based on lesson materials, automating document summaries and quiz generation, saving 70% of preparation time for tutors.
