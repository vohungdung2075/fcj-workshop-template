---
title: "Proposal"
date: 2026-06-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# LearnSphere - Online Learning Platform Integrated with AI
## An AWS Solution for Next-Gen E-Learning & AI Integration

### 1. Executive Summary

LearnSphere is proposed as an online learning platform for students, tutors, and administrators. It centralizes courses, lessons, videos, documents, quizzes, learning progress, and communication in one application. The AI Assistant allows students to ask questions using lesson context, summarizes learning documents, and helps tutors generate quiz questions at a selected difficulty.

The solution uses a hybrid web architecture on AWS. The React/Vite Frontend is compiled into static assets, stored in a private Amazon S3 bucket, and delivered through Amazon CloudFront. The `www.learnsphere.id.vn` domain is managed through Tenten DNS, uses a CNAME pointing to CloudFront, and is protected by an HTTPS certificate issued through AWS Certificate Manager. The Express.js Backend is packaged as a Docker image, stored in Amazon ECR, and runs on Amazon EC2. Course media is stored in a separate private S3 bucket and is accessible only through expiring presigned URLs. MongoDB Atlas stores application data, while the Groq API provides AI inference for chat, document summarization, and quiz generation. GitHub Actions automatically tests, builds, and deploys the system through AWS OIDC and Systems Manager.

The MVP prioritizes a student-project budget, delivery within the internship schedule, private resource access, and convenient HTTPS access through a custom domain. As usage grows, the architecture can introduce AWS WAF, an Application Load Balancer, Auto Scaling, Secrets Manager, and asynchronous media processing.

### 2. Problem Statement

Online learning activities are often fragmented across video platforms, file-sharing tools, quiz applications, and independent AI chatbots. This creates several problems:

* Students cannot easily track courses, progress, and quiz performance in one place.
* Traditional e-learning platforms provide limited contextual support outside scheduled class time, while general-purpose chatbots may answer without grounding their responses in lesson materials.
* Tutors spend significant time reading documents, preparing summaries, authoring questions, handling enrollment, and reviewing individual students.
* Content from PDF, DOCX, and scanned documents is not automatically converted into reusable lesson context.
* Large videos and documents are inefficient when uploaded through the Backend server and can consume its network and compute resources.
* Administrators lack centralized metrics for users, courses, traffic, and storage.
* Manual deployments and fragmented logs are vulnerable to configuration errors, downtime, delayed incident detection, and difficult rollbacks.

LearnSphere addresses these issues through a unified role-based system with secure media storage, document-grounded AI, quizzes and learning reports, and an AWS deployment and monitoring workflow.

### 3. Objectives and Scope

#### 3.1. Objectives

* Deliver a responsive SPA accessible through `www.learnsphere.id.vn` over HTTPS.
* Support clearly separated `student`, `tutor`, and `admin` roles.
* Manage the complete lifecycle of courses, lessons, enrollment, progress, and quizzes.
* Upload large videos, documents, thumbnails, and avatars directly to S3 securely.
* Integrate AI chat, document summaries, and AI-assisted quiz generation.
* Track model IDs and token usage and enforce per-user AI request limits.
* Automatically test, build, and deploy the application from GitHub to AWS.
* Monitor Backend health and infrastructure metrics and alert administrators when EC2 fails.

#### 3.2. MVP Scope

| Role | Main capabilities |
| --- | --- |
| Student | Enroll in courses, access lessons and resources, track progress, attempt quizzes, review results, use the AI Assistant, receive notifications, and join discussions |
| Tutor | Create and manage courses, lessons, media, and quizzes; review enrollment requests; generate AI questions; inspect student progress and detailed quiz attempts |
| Administrator | Manage user accounts and statuses; inspect courses and quizzes by tutor; monitor platform statistics and S3 storage |
| Platform | JWT authentication, authorization, notifications, discussions, presigned and multipart uploads, orphan cleanup, AI rate limiting, CI/CD, and monitoring |

#### 3.3. Out of Scope

Course payments, real-time virtual classrooms, a native mobile application, video-content summarization, personalized recommendations, Multi-AZ deployment, and Auto Scaling are outside the current MVP. They are treated as future enhancements.

### 4. Proposed Solution Architecture

![LearnSphere system architecture](/images/LEARNSHPHERE.png)

**Figure 1. LearnSphere production architecture and service interactions.**

The diagram presents the runtime, deployment, storage, AI, and monitoring paths. CloudFront and IAM are global AWS services.\ 
ECR, Systems Manager, S3, EC2, CloudWatch, and SNS operate in `ap-southeast-1`.\
The VPC contains an Internet Gateway, one Availability Zone, a public subnet, and the EC2 Backend.\
GitHub, users, MongoDB Atlas, Groq, and the administrator's email service remain outside AWS Cloud.

#### 4.1. Application Access Flow

1. The browser resolves `www.learnsphere.id.vn` through Tenten DNS to CloudFront.
2. The user connects over HTTPS, and CloudFront presents the TLS certificate managed by ACM.
3. CloudFront retrieves the Frontend from the private S3 bucket through Origin Access Control.
4. Requests matching `/api/*` are forwarded to the EC2 Backend without caching.
5. The Backend validates the JWT, checks role permissions, and queries MongoDB Atlas.
6. For media operations, the Backend only issues presigned URLs; the browser uploads to or downloads from the private S3 bucket directly.

#### 4.2. AI and Document Flow

1. A tutor uploads a PDF or DOCX document to S3 and attaches its object key to a lesson.
2. The Backend downloads the authorized object and extracts text with `pdf-parse` or Mammoth.
3. Scanned PDFs are processed sequentially with Vietnamese Tesseract.js OCR and resource limits.
4. Normalized content is used for summaries, lesson-grounded chat, and quiz generation.
5. Summaries are persisted to prevent repeated inference, while model IDs and token usage are recorded.
6. The Backend sends the prepared context to Groq and handles provider throttling, timeout, and malformed-response errors explicitly.

#### 4.3. CI/CD Flow

1. A push to `main` triggers GitHub Actions.
2. The Backend installs dependencies, runs tests, builds a Docker image, and pushes the commit-SHA image to ECR.
3. GitHub Actions obtains temporary AWS credentials through OIDC and sends deployment commands through Systems Manager.
4. EC2 runs a candidate container and checks `/health/ready` before replacing production; the previous container is temporarily retained for rollback.
5. The Frontend is built and synchronized to S3, followed by a CloudFront invalidation after successful Backend deployment.

### 5. Component Design

| Component | Technology/Service | Responsibility |
| --- | --- | --- |
| Frontend | React 18, TypeScript, Vite, Tailwind CSS, KaTeX | Role-based SPA, direct uploads, quizzes, AI Assistant, and mathematical rendering |
| Backend | Node.js, Express 5, Docker | REST API, authentication, business logic, signed URLs, AI orchestration, and health checks |
| Database | MongoDB Atlas, Mongoose | Users, courses, lessons, enrollment, quizzes, attempts, progress, notifications, AI messages, and cleanup state |
| Storage | Two private Amazon S3 buckets | Static Frontend assets; videos, documents, thumbnails, and avatars |
| AI | Groq API | Chat, document summaries, and quiz generation |
| Document processing | pdf-parse, Mammoth, Tesseract.js | PDF/DOCX extraction and Vietnamese scanned-PDF OCR |
| Compute | Amazon EC2, Amazon Linux 2023 | Dockerized Backend and OCR workloads |
| Container registry | Amazon ECR | Commit-SHA Backend images |
| Delivery | Amazon CloudFront and CloudFront Function | HTTPS, SPA CDN, route fallback, and `/api/*` forwarding |
| Domain and TLS | Tenten DNS and AWS Certificate Manager | Point the `www` CNAME to CloudFront, validate domain ownership, and provide HTTPS |
| CI/CD | GitHub Actions, AWS OIDC, Systems Manager | Test, build, deploy, health verification, and rollback |
| Observability | Amazon CloudWatch and SNS | Container logs, CPU/status alarms, and email notifications |

### 6. Technical Implementation and Security

#### 6.1. Primary Controls

* JWTs are stored in HttpOnly cookies; production uses Secure cookies and a CORS allowlist.
* `bcryptjs`, Helmet, request rate limits, JSON body limits, and authorization middleware protect the API.
* Both S3 buckets remain private with Block Public Access enabled.
* CloudFront accesses the Frontend bucket through OAC; media is accessible only through short-lived presigned URLs.
* `www.learnsphere.id.vn` points to CloudFront through a CNAME; ACM manages the TLS certificate, and its DNS validation record is retained for renewal.
* Large videos use multipart upload with tracked `UploadSession` records, abort/retry support, and orphan cleanup.
* Old avatars, thumbnails, videos, or documents are deleted only after the database successfully references the replacement.
* EC2 uses an IAM role, while GitHub uses OIDC. Long-term AWS access keys are not stored on the server or in the repository.
* The EC2 Security Group allows the Backend port only from the AWS-managed CloudFront origin-facing prefix list.
* AI operations use per-user rate limits, timeouts, standardized errors, summary caching, and token accounting.
* CloudWatch monitors `CPUUtilization` and `StatusCheckFailed`, while SNS sends alarm emails.

#### 6.2. MVP Architecture Limitations

The Backend currently runs on a single EC2 instance in one Availability Zone, which remains a single point of failure. Application secrets are protected in a permission-restricted `.env` file but are not yet centrally managed by Secrets Manager. MongoDB Atlas and Groq are external services, so the system depends on Internet connectivity and third-party availability.

### 7. Implementation Plan and Milestones

| Phase | Schedule | Activities | Milestone |
| --- | --- | --- | --- |
| 1. AWS foundation | Weeks 1–4 | Cloud, IAM, cost management, VPC, EC2, S3, databases, CloudFront, Docker, ECR, CloudWatch, SNS, generative AI, and CI/CD | Foundational knowledge and architecture completed |
| 2. LearnSphere Backend | Week 5 | Authentication, courses, lessons, enrollment, quizzes, progress, notifications, and S3 cleanup | Core business APIs completed |
| 3. AI and documents | Week 6 | AI Provider, chat history, rate limits, PDF/DOCX indexing, OCR, summaries, AI quizzes, and multipart cleanup | Grounded AI and reliable uploads completed |
| 4. Frontend and testing | Week 7 | Role-based UI, course/lesson/quiz management, student reports, and testing | User workflows completed |
| 5. Production deployment | Week 8 | S3, CloudFront, ECR, EC2, IAM, OIDC, CI/CD, CloudWatch, SNS, ACM, and the custom domain | LearnSphere operational at `www.learnsphere.id.vn` |
| 6. Validation and report | Week 9 | End-to-end testing, screenshots, architecture, bilingual workshop, and demonstration | Project handover and report completed |

### 8. Budget Estimation

Assumptions: one `t3.small` EC2 instance running continuously, approximately 8–10 GB of EBS storage, 10 GB of S3 media, less than 100 GB of monthly delivery, less than 1 GB of ECR images, low log volume, and a test workload of approximately 50 users.

| Component | Estimated monthly cost (USD) | Notes |
| --- | ---: | --- |
| EC2 `t3.small` | 18–22 | Depends on Singapore pricing and running hours |
| EBS and public IPv4 | 4–6 | Root volume and public IPv4 address |
| Amazon S3 | 0.3–1.5 | 10 GB storage, requests, and lifecycle |
| Amazon CloudFront | 0–3 | Small workloads may remain within eligible free allowances |
| AWS Certificate Manager | 0 | Non-exportable public certificate used with CloudFront |
| Amazon ECR | 0–0.2 | Images limited by a lifecycle policy |
| Amazon CloudWatch and SNS | 0–2 | Logs, two standard alarms, and email notifications |
| MongoDB Atlas M0 | 0 | External service using its free tier |
| Groq | 0 or plan-dependent | External AI provider used by the application |
| Tenten domain and DNS | Registration-dependent | External service; the domain has already been registered |
| **Estimated AWS total** | **23–35/month** | Excludes taxes, Groq, Tenten domain fees, and high traffic |

This range is a learning-environment estimate rather than a fixed invoice. Actual AWS charges depend on the Region, traffic, video usage, and the account’s Free Tier or credits; Groq usage is billed separately according to the selected external plan. Reference pricing is available for [Amazon EC2 On-Demand](https://aws.amazon.com/ec2/pricing/on-demand/), [Amazon S3](https://aws.amazon.com/s3/pricing/), and [Amazon CloudFront](https://aws.amazon.com/cloudfront/pricing/). Before scaling, the workload should be recalculated with the [AWS Pricing Calculator](https://calculator.aws/) and protected with an AWS Budget.

### 9. Risk Assessment

| Risk | Impact | Probability | Mitigation |
| --- | --- | --- | --- |
| Groq API throttling, quota exhaustion, or malformed responses | High | Medium | Per-user limits, summary caching, bounded prompts, response validation, timeouts, retries, and explicit errors |
| Single EC2 failure | High | Low–Medium | Health checks, status alarms, deployment rollback; future Multi-AZ ALB and Auto Scaling |
| Slow or interrupted large-video upload | Medium | Medium | Presigned multipart upload, progress, retry/abort, cleanup; future media CDN and HLS |
| High OCR memory or CPU usage for scanned PDFs | Medium | High | Page and image limits, sequential OCR, timeout, single concurrency, and run IDs |
| Orphaned S3 objects increase cost | Medium | Medium | Safe deletion, cleanup queue, orphan upload cleanup, and lifecycle rules |
| Incorrect DNS, certificate, CORS, cookie, or CloudFront behavior | High | Medium | Retain the ACM validation CNAME, use `www` as the canonical origin, keep same-origin `/api`, apply the production allowlist, and separate cache behaviors |
| Incorrect OIDC or IAM configuration | High | Medium | Repository-restricted trust policy, least privilege, and no long-term access keys |
| Exposure of secrets in `.env` | High | Low | File permission `600`, no repository commit; future Secrets Manager/Parameter Store |
| MongoDB Atlas or Groq outage | Medium | Low–Medium | Ready checks, timeout handling, error logs, backups, and alternative providers |
| Increasing AI and storage cost | Medium | Medium | Token accounting, AI rate limits, cached results, S3 cleanup, AWS Budgets, and CloudWatch |

### 10. Expected Outcomes

#### 10.1. Functional Outcomes

* A complete online learning platform for students, tutors, and administrators.
* Centralized course, lesson, enrollment, progress, quiz, notification, and discussion management.
* Secure S3 media transfer without proxying large files through the Backend.
* Lesson-grounded AI chat, document summarization, and AI-assisted quiz generation.
* Tutor access to individual student progress and detailed assessment results.
* Administrator access to account, content, and platform metrics.

#### 10.2. Technical Outcomes

* Frontend and Backend deployed on AWS and accessible through `https://www.learnsphere.id.vn`.
* Automated CI/CD using temporary credentials, health validation, and safe rollback.
* Private storage, least-privilege IAM, secure cookies, and request rate limiting.
* Logs, health endpoints, platform metrics, CloudWatch alarms, and SNS notifications.
* An architecture that can evolve without rewriting the complete application.

#### 10.3. Operational Value and Evaluation Metrics

* Document extraction, reusable summaries, and AI-generated quiz drafts reduce repetitive preparation work, while tutors remain responsible for reviewing generated content.
* The lesson-grounded AI Assistant gives students on-demand support beyond scheduled class time, subject to Groq availability and configured request limits.
* Direct S3 uploads prevent large media files from being proxied through EC2, while CloudFront improves static-content delivery.
* Automated CI/CD makes deployments repeatable, validates Backend health, and keeps a rollback path when a new container fails.
* Benefits should be evaluated with measured baselines such as quiz-preparation time, deployment duration, AI success rate, monthly infrastructure cost, and incident-recovery time. Percentage savings and payback periods should only be reported after sufficient production measurements are available.

### 11. Future Development

* Add AWS WAF in front of CloudFront.
* Place the Backend behind an Application Load Balancer and a Multi-AZ Auto Scaling Group.
* Move secrets to AWS Secrets Manager or Systems Manager Parameter Store.
* Introduce a CloudFront media distribution, HLS, and AWS Elemental MediaConvert for video.
* Move OCR and indexing to an asynchronous SQS queue and dedicated workers.
* Add SES for production email, automated backups, and Infrastructure as Code.
* Explore retrieval-augmented generation (RAG), semantic search, and personalized learning paths.
