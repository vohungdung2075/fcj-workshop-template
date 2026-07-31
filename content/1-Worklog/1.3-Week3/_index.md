---
title: "Week 3 Worklog"
date: 2026-06-15
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 objectives

* Understand how AWS provides and manages relational databases through Amazon RDS.
* Learn the fundamental NoSQL database concepts with Amazon DynamoDB.
* Understand how Amazon CloudFront distributes content and improves application access speed.
* Learn how to package applications with Docker and store Docker images in Amazon ECR.
* Practice combining AWS services to prepare for application deployment in the following weeks.

### Tasks completed during the week

| Day | Tasks | Start date | Completion date | Reference material |
| --- | --- | --- | --- | --- |
| **2** | - Studied **Amazon Relational Database Service (RDS)**.<br>- Explored its primary components and concepts:<br>&emsp;+ Database engines<br>&emsp;+ DB instances<br>&emsp;+ DB instance classes<br>&emsp;+ Storage<br>&emsp;+ Security groups<br>&emsp;+ Automated backups<br>&emsp;+ Multi-AZ deployments<br>- Learned how applications securely connect to an RDS database within a VPC. | 06/15/2026 | 06/15/2026 | https://000005.awsstudygroup.com/ |
| **3** | - Studied the NoSQL database model and **Amazon DynamoDB**.<br>- Explored the fundamental DynamoDB components:<br>&emsp;+ Tables and items<br>&emsp;+ Attributes<br>&emsp;+ Partition keys<br>&emsp;+ Sort keys<br>&emsp;+ Capacity modes<br>- Created a table and practiced basic create, read, update, and delete operations. | 06/16/2026 | 06/16/2026 | https://000060.awsstudygroup.com/ |
| **4** | - Studied **Amazon CloudFront** and the role of a Content Delivery Network (CDN).<br>- Explored the main concepts:<br>&emsp;+ Distributions<br>&emsp;+ Origins<br>&emsp;+ Edge locations<br>&emsp;+ Cache behaviors<br>&emsp;+ Cache invalidations<br>- Created a CloudFront distribution with an S3 bucket as its origin.<br>- Learned how to restrict direct S3 access and deliver content through HTTPS. | 06/17/2026 | 06/17/2026 | https://000094.awsstudygroup.com/ |
| **5** | - Studied **Docker** fundamentals and the role of containers in software development.<br>- Distinguished between Docker images, containers, and registries.<br>- Learned the Dockerfile structure and its basic instructions:<br>&emsp;+ FROM<br>&emsp;+ WORKDIR<br>&emsp;+ COPY<br>&emsp;+ RUN<br>&emsp;+ EXPOSE<br>&emsp;+ CMD<br>- Created a Dockerfile, built an image, and ran a sample application in a container. | 06/18/2026 | 06/18/2026 | https://000015.awsstudygroup.com/ |
| **6** | - Studied **Amazon Elastic Container Registry (ECR)** and the Docker image management workflow.<br>- Practiced:<br>&emsp;+ Creating an ECR repository<br>&emsp;+ Authenticating Docker with ECR<br>&emsp;+ Tagging a Docker image<br>&emsp;+ Pushing an image to ECR<br>&emsp;+ Pulling an image from ECR<br>- Ran a container from an image stored in ECR and reviewed the application packaging, storage, and distribution workflow. | 06/19/2026 | 06/19/2026 | https://000017.awsstudygroup.com/ |

### Week 3 achievements

* Understood the role of Amazon RDS and its fundamental components:
  * Database engines
  * DB instances
  * Storage
  * Security groups
  * Automated backups
  * Multi-AZ deployments

* Learned the Amazon DynamoDB concepts of tables, items, attributes, partition keys, and sort keys.
* Understood how Amazon CloudFront uses edge locations and caching to distribute content.
* Created a CloudFront distribution with Amazon S3 as its origin.
* Distinguished between Docker images, containers, and registries.
* Created a Dockerfile, built a Docker image, and ran a sample application in a container.
* Created an Amazon ECR repository and completed the authentication, tagging, push, and pull workflow for Docker images.
* Built a foundation in databases, content delivery, and containers to prepare for deploying applications on AWS.
