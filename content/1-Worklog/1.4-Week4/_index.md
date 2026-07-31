---
title: "Week 4 Worklog"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 objectives

* Understand how to monitor resources, collect logs, and create alerts with Amazon CloudWatch.
* Learn how to use Amazon SNS to send notifications when system incidents occur.
* Explore Amazon Bedrock and the integration of foundation models into applications.
* Understand how GitHub Actions automates source code testing, packaging, and deployment.
* Combine the services learned during the first four weeks to design an AWS web application architecture.

### Tasks completed during the week

| Day | Tasks | Start date | Completion date | Reference material |
| --- | --- | --- | --- | --- |
| **2** | - Studied **Amazon CloudWatch** and its role in system monitoring.<br>- Explored the main components:<br>&emsp;+ Metrics<br>&emsp;+ Logs and log groups<br>&emsp;+ Dashboards<br>&emsp;+ Alarms<br>- Monitored EC2 metrics such as CPUUtilization, NetworkIn, and NetworkOut.<br>- Created a CloudWatch dashboard to display fundamental monitoring information. | 06/22/2026 | 06/22/2026 | https://000008.awsstudygroup.com/ |
| **3** | - Studied **Amazon Simple Notification Service (SNS)** and the publish/subscribe model.<br>- Explored the main components:<br>&emsp;+ Topics<br>&emsp;+ Publishers<br>&emsp;+ Subscriptions<br>&emsp;+ Endpoints<br>- Created an SNS topic, subscribed an email endpoint, and confirmed the subscription.<br>- Connected a CloudWatch alarm to SNS to send a notification when a metric exceeded its configured threshold. | 06/23/2026 | 06/23/2026 | https://000077.awsstudygroup.com/ |
| **4** | - Studied **Amazon Bedrock** and the concept of foundation models.<br>- Explored the Model Catalog, Bedrock Playground, and inference options.<br>- Learned how an application sends prompts and receives model responses through an API.<br>- Studied the IAM permissions required for model invocation and the selection of Regions, model IDs, and inference profiles.<br>- Learned how tokens, quotas, and throttling affect AI feature costs and availability. | 06/24/2026 | 06/24/2026 | https://000056.awsstudygroup.com/ |
| **5** | - Studied the **Continuous Integration and Continuous Deployment (CI/CD)** process in software development.<br>- Became familiar with GitHub Actions and the YAML workflow structure:<br>&emsp;+ Triggers<br>&emsp;+ Jobs<br>&emsp;+ Steps<br>&emsp;+ Secrets and variables<br>- Explored the process of building, testing, and packaging an application as a Docker image.<br>- Learned how GitHub Actions authenticates with AWS through OpenID Connect (OIDC) instead of storing long-term access keys. | 06/25/2026 | 06/25/2026 | https://000051.awsstudygroup.com/ |
| **6** | - Reviewed the AWS services studied during the first four weeks.<br>- Designed a sample architecture for a web application consisting of:<br>&emsp;+ Amazon S3 and CloudFront for Frontend delivery<br>&emsp;+ Amazon EC2 running a Dockerized Backend<br>&emsp;+ Amazon ECR for Docker image storage<br>&emsp;+ Amazon CloudWatch and SNS for monitoring and alerts<br>&emsp;+ IAM and VPC for access control and networking<br>- Evaluated the architecture in terms of security, performance, reliability, and cost.<br>- Prepared the plan for transitioning to the LearnSphere project development phase. | 06/26/2026 | 06/26/2026 | https://cloudjourney.awsstudygroup.com/ |

### Week 4 achievements

* Understood the main Amazon CloudWatch monitoring components:
  * Metrics
  * Logs
  * Dashboards
  * Alarms

* Monitored fundamental EC2 metrics and created a CloudWatch dashboard.
* Created an SNS topic and email subscription and connected SNS to a CloudWatch alarm.
* Understood the role of Amazon Bedrock in integrating generative AI into applications.
* Learned the factors required for model invocation:
  * Region
  * Model ID
  * Inference profile
  * IAM permissions
  * Tokens and quotas
* Understood the basic GitHub Actions workflow structure, including triggers, jobs, and steps.
* Learned the process of building, testing, creating a Docker image, and automatically deploying an application.
* Understood the benefits of using OIDC to allow GitHub Actions to access AWS without long-term access keys.
* Completed a sample architecture diagram combining S3, CloudFront, EC2, ECR, CloudWatch, SNS, IAM, and VPC.
* Completed the foundational AWS learning phase and prepared to begin developing the LearnSphere project.
