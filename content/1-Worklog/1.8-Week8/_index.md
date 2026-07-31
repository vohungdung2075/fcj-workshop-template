---
title: "Week 8 Worklog"
date: 2026-07-20
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 objectives

* Complete production hardening, testing, and configuration for LearnSphere.
* Deploy the Backend, Frontend, and media storage to AWS services.
* Implement automated CI/CD with GitHub Actions and AWS OIDC.
* Validate application features in the production environment.
* Configure monitoring and alerts and complete the system architecture diagram.

### Tasks completed during the week

| Day | Tasks | Start date | Completion date | Reference material |
| --- | --- | --- | --- | --- |
| **2** | - Reviewed the entire project before deployment.<br>- Added environment validation and production configuration.<br>- Configured HttpOnly/Secure cookies, Helmet, a CORS allowlist, rate limits, and JSON request limits.<br>- Completed 404 and global error handlers, graceful shutdown, and live/ready health checks.<br>- Fixed Mongoose warnings, removed unused files, and ran Backend tests and the Frontend production build. | 07/20/2026 | 07/20/2026 | https://expressjs.com/en/advanced/best-practice-security.html<br>https://docs.docker.com/guides/nodejs/ |
| **3** | - Prepared the AWS infrastructure for LearnSphere.<br>- Created private S3 buckets for the Frontend and media and configured CloudFront Origin Access Control.<br>- Created an Amazon ECR repository and an EC2 instance running Docker.<br>- Attached an IAM role to EC2 for required ECR, S3, CloudWatch, and Bedrock access.<br>- Configured MongoDB Atlas, the security group, environment variables, and verified Docker on EC2. | 07/21/2026 | 07/21/2026 | https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html<br>https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html<br>https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html |
| **4** | - Completed the automated GitHub Actions deployment workflow.<br>- Configured GitHub OIDC and an IAM role for AssumeRoleWithWebIdentity.<br>- Built, tested, and tagged Docker images by commit SHA and pushed them to Amazon ECR.<br>- Automatically updated the Backend container on EC2 and checked its health endpoint.<br>- Built the Frontend, synchronized files to S3, and invalidated the CloudFront cache. | 07/22/2026 | 07/22/2026 | https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws<br>https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html |
| **5** | - Validated LearnSphere through the CloudFront HTTPS domain.<br>- Tested registration, login, authorization, course, lesson, and enrollment management.<br>- Tested thumbnail, avatar, document, and multipart video uploads through presigned URLs.<br>- Tested video playback, document summaries, AI chat, quiz generation, quiz attempts, and result views.<br>- Resolved OIDC, CORS, and AI response-structure issues and confirmed that the Backend container was healthy. | 07/23/2026 | 07/23/2026 | https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html<br>https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html |
| **6** | - Completed the statistics API and System Monitoring page for administrators.<br>- Configured Amazon CloudWatch alarms for CPUUtilization and StatusCheckFailed.<br>- Created an Amazon SNS topic and email subscription for alerts.<br>- Created the system architecture diagram and documented the flow between CloudFront, S3, EC2, ECR, IAM, CloudWatch, SNS, MongoDB Atlas, and the AI provider. | 07/24/2026 | 07/24/2026 | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating_status_check_alarms.html<br>https://docs.aws.amazon.com/sns/latest/dg/welcome.html<br>https://aws.amazon.com/architecture/icons/ |

### Week 8 achievements

* Completed Backend security checks, configuration validation, error handling, and health checks.

* Successfully completed Backend tests and the Frontend production build.
* Deployed the Dockerized Backend to EC2 with images stored in Amazon ECR.
* Deployed the Frontend to a private S3 bucket and delivered it through Amazon CloudFront HTTPS.
* Configured IAM roles for EC2 and GitHub Actions without storing long-term AWS access keys.
* Completed GitHub Actions CI/CD for both the Backend and Frontend.
* Validated the main features in the production environment.
* Confirmed that the Backend container was healthy and connected to MongoDB.
* Completed the System Monitoring page with system and S3 storage metrics.
* Configured CloudWatch alarms and Amazon SNS email notifications.
* Completed the architecture diagram and documented the operational flow between the LearnSphere system components.
