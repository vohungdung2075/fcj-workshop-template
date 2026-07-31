---
title: "Blogs Posted"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Below is the list of technical blog posts published on the [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj):

### [Blog 1 - Why Our Team Didn't Use Amazon Bedrock Knowledge Bases for Our RAG Chatbot](3.1-Blog1/)
This article analyzes why our team chose to build a custom RAG pipeline with FAISS instead of adopting the fully managed Knowledge Bases for Amazon Bedrock. It evaluates the trade-offs between rapid managed deployments and total ingestion pipeline control (custom OCR via Amazon Textract, tailored chunking, metadata management, and multi-tenant data isolation).

---

### [Blog 2 - [AWS Architecture] System Architecture Analysis of AI Learning Platform (LearnSphere)](3.2-Blog2/)
A detailed analysis of the frontend and backend cloud architecture for the LearnSphere AI platform. The architecture combines Serverless Static Hosting (S3 + CloudFront), Containerized Backend (EC2 + ECR), automated CI/CD via GitHub Actions, CloudWatch observability, and managed integrations (OpenAI API, MongoDB Atlas).

---

### [Blog 3 - [AWS Security & Network] Network Segmentation & Infrastructure Security Analysis for AI Learning Platform](3.3-Blog3/)
An in-depth analysis of cloud network architecture (VPC, Public Subnets, IGW in Singapore Region), S3 storage isolation strategies (Static Frontend vs. AI Media Data buckets), secure outbound traffic management (Parameter Store, Secrets Manager, MongoDB Atlas IP Whitelisting), CloudWatch observability, and future security upgrade roadmaps (ALB/WAF).