---
title: "Project overview"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### LearnSphere introduction

LearnSphere is an online learning platform for students, tutors, and administrators. It brings courses, lessons, videos, documents, quizzes, progress tracking, and contextual AI assistance into one application.

#### Background and problem

Conventional online learning systems often separate content, assessments, and progress tracking. Tutors spend significant time reading documents, composing questions, and reviewing results manually, while students cannot always receive timely assistance during self-study. Routing large videos and documents through the Backend also increases server load and complicates scaling.

LearnSphere addresses these concerns by:

* Centralizing courses, lessons, students, quizzes, and progress.
* Storing media in Amazon S3 and transferring files directly through presigned URLs.
* Using document content as context for AI assistance, summarization, and quiz generation.
* Separating Frontend, Backend, data, and media so each layer can operate independently.
* Automating production releases through GitHub Actions and AWS.

#### Project source code

![LearnSphere GitHub repository](/images/learnsphere-github-repository.png)

*Figure 5.1. The LearnSphere repository contains the Frontend, Backend, CI/CD workflow, and deployment documentation.*

Production source is maintained in [HoiaeKHMT/LearnSphere](https://github.com/HoiaeKHMT/LearnSphere). The `main` branch is the production release source; GitHub Actions validates, packages, and deploys each commit through the configured workflow.

#### Workshop scope

| Area | Included |
| --- | --- |
| Application | React/Vite Frontend and Express/Docker Backend |
| Infrastructure | Multi-AZ VPC, ALB, ASG, private EC2, NAT Gateways |
| Storage | S3 Frontend, S3 Media, and ECR |
| Delivery | CloudFront, HTTPS, ACM, and custom domain |
| Data and AI | MongoDB Atlas, document/OCR processing, and Groq |
| Operations | GitHub OIDC, CI/CD, CloudWatch Logs/Alarms, and SNS |

The workshop focuses on the production architecture deployed in `ap-southeast-1`. It does not repeat the implementation of every application screen; it explains how to deploy the completed application to AWS and validate the result.

#### Technology stack

| Component | Technology | Responsibility |
| --- | --- | --- |
| Frontend | React 18, TypeScript, Vite, Tailwind CSS | Role-based SPA, courses, quizzes, uploads, and AI Assistant |
| Backend | Node.js 24, Express 5, Docker | REST API, authentication, business logic, presigned URLs, and AI orchestration |
| Database | MongoDB Atlas, Mongoose | Users, courses, lessons, enrolments, quizzes, attempts, progress, and AI messages |
| Media | Amazon S3 | Videos, documents, thumbnails, and avatars |
| AI | Groq API | Lesson-grounded chat, document summaries, and quiz generation |
| Documents | pdf-parse, Mammoth, Tesseract.js | PDF/DOCX extraction and scanned-PDF OCR |

#### Main capabilities

* Students enrol in courses, access lessons and documents, track progress, and complete quizzes.
* Tutors manage courses, lessons, students, quizzes, and detailed learning results.
* Administrators manage accounts, monitor the platform, and inspect data by tutor.
* Large media files use direct presigned and multipart uploads.
* AI uses document content as context, while persisted summaries prevent repeated inference.

#### Technical objectives

1. Host a private Frontend in S3 and deliver it over HTTPS through CloudFront.
2. Run the Backend on at least two private EC2 instances across two AZs.
3. Accept Backend traffic only from the ALB Security Group.
4. Maintain Backend capacity through an ASG and `/health/ready` health checks.
5. Release immutable images identified by Git commit SHA.
6. Avoid long-lived AWS access keys in the repository and on servers.
7. Provide centralized logs, alerts, and rollback behavior.

#### Repository organization

```text
LearnSphere/
├── LearnSphere_BE/          # Express API, models, services, Dockerfile
├── LearnSphere_FE/          # React/Vite SPA
├── .github/workflows/       # Production CI/CD
├── docs/                    # Deployment documentation
└── README.md
```

#### Production application

![LearnSphere production homepage](/images/learnsphere-production-homepage.png)

*Figure 5.2. LearnSphere running on the production custom domain.*

After completing the workshop, the reader can:

* Reproduce the LearnSphere Multi-AZ architecture.
* Release a new version through `git push`.
* Verify two ASG instances and two healthy Target Group targets.
* Validate Frontend, API, S3 Media, MongoDB, and Groq end to end.
* Access the application at [https://www.learnspherev2.id.vn](https://www.learnspherev2.id.vn).
