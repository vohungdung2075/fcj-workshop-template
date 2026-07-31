---
title: "Workshop"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploying LearnSphere on AWS

#### Overview

This workshop documents the complete journey from local LearnSphere source code to production in the AWS Singapore Region (`ap-southeast-1`). The React/Vite Frontend is delivered through Amazon S3 and CloudFront. The Dockerized Node.js/Express Backend runs in an Auto Scaling Group with two private EC2 instances across two Availability Zones and is exposed through an Application Load Balancer.

The current CI/CD process uses GitHub OIDC, ECR, Systems Manager Parameter Store, and Auto Scaling Instance Refresh with health validation and image-tag rollback.

#### Key outcomes

* Production website: [https://www.learnspherev2.id.vn](https://www.learnspherev2.id.vn).
* Backend instances run in two private subnets across two Availability Zones.
* The ALB distributes API requests to two healthy targets.
* The ASG maintains `min=2`, `desired=2`, and `max=4`.
* Frontend and media objects are stored in separate private S3 buckets.
* MongoDB Atlas and Groq are reached through per-AZ NAT Gateways.
* GitHub Actions uses temporary OIDC credentials instead of long-lived AWS keys.
* CloudWatch centralizes logs and SNS notifies the administrator.

#### Contents

1. [Project overview](5.1-overview/)
2. [Deployment preparation](5.2-preparation/)
3. [Architecture and system flows](5.3-architecture/)
4. [Core AWS infrastructure](5.4-infrastructure/)
5. [High-availability Backend deployment](5.5-backend-ha/)
6. [Frontend, CloudFront, and domain deployment](5.6-frontend-domain/)
7. [CI/CD automation](5.7-cicd/)
8. [Data, media, and AI](5.8-data-ai/)
9. [Monitoring and alerting](5.9-monitoring/)
10. [Testing and results](5.10-testing/)
11. [Cost analysis](5.11-cost/)
12. [Resource cleanup](5.12-cleanup/)

#### Conclusion

This workshop completes the journey from LearnSphere source code to a secure, automated, and highly available production environment. The system combines CloudFront, S3, ALB, and an Auto Scaling Group across two Availability Zones; GitHub Actions, GitHub OIDC, and ECR release the Backend without SSH administration or long-lived access keys. The result is an online learning platform that can operate reliably, provide centralized observability, and scale further as user demand grows.
