---
title: "Data, media storage, and AI"
date: 2026-07-31
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

#### Objectives

Section 5.8 describes application data flows after infrastructure deployment: shared business data in MongoDB Atlas, learning files in S3, document extraction/OCR, and Groq-powered AI operations.

Primary objectives:

* All backend EC2 instances share one durable database.
* Large files do not pass through EC2 memory or local storage.
* Uploads use presigned URLs, multipart transfer, and session tracking.
* PDF, DOCX, and scanned PDF documents become bounded knowledge text.
* AI chat, summaries, and quizzes use the correct course context.
* Model and token usage are persisted.
* Rate limits and explicit errors protect provider quota.

#### Data classification

| Data | Storage | Examples |
| --- | --- | --- |
| Business data | MongoDB Atlas | user, course, lesson, enrollment, quiz, attempt |
| Media/learning files | `learnsphere-media-2` | thumbnail, avatar, video, PDF, DOCX |
| Runtime configuration | SSM Parameter Store | backend environment, image tag |
| AI-derived data | MongoDB Atlas | indexed text, summary, history, token usage |

#### Detailed flows

1. [MongoDB Atlas and the shared data model](5.8.1-mongodb-data/)
2. [Presigned URLs, multipart upload, and orphan cleanup](5.8.2-s3-media-upload/)
3. [Document extraction and OCR](5.8.3-document-ocr/)
4. [Groq AI, context, rate limiting, and usage](5.8.4-groq-ai/)

#### Data boundaries

```text
Business metadata and state → MongoDB Atlas
Binary files                → S3 Media
Extracted text              → Lesson AI index
Selected prompt and context → Groq API
Response and token usage    → MongoDB Atlas
```

Video binaries are never sent to the LLM. AI uses extracted document text and bounded lesson context to reduce latency, cost, and context overflow.

#### Future Roadmap & Architectural Evolution

To transition LearnSphere into a fully **AWS Native** platform while optimizing overall performance and security, our team plans to upgrade two core components in future releases:

1. **Database Migration: From MongoDB Atlas to Amazon DynamoDB (AWS Native NoSQL)**
   - **Rationale**: Eliminate cross-cloud network round-trips outside AWS (reducing latency), leveraging AWS's Fully Managed NoSQL service with seamless auto-scaling and high availability.
   - **Benefits**: Deep integration with **AWS IAM Policies**, **AWS KMS (Encryption at rest)**, **CloudWatch Metrics**, and **VPC Gateway Endpoints** (free internal access bypassing the internet).
   - **Migration Approach**: Utilize the **`Dynamoose`** ODM library to migrate Mongoose Schemas (`User`, `Course`, `Lesson`, `QuizAttempt`...) to DynamoDB Tables rapidly without heavy refactoring of backend controller logic.

2. **AI Engine Migration: From Groq API to Amazon Bedrock (AWS Fully Managed Generative AI)**
   - **Rationale**: Bring all GenAI processing natively into the AWS ecosystem, adhering strictly to enterprise data privacy standards without exposing data to third-party endpoints.
   - **Benefits**: Access state-of-the-art Foundation Models (Anthropic Claude 3.5 Sonnet, Amazon Titan, Llama 3) via a single unified API; natively leverage **Amazon Bedrock Guardrails** (content moderation), **Bedrock Knowledge Bases** (managed RAG), and **Bedrock Agents** for advanced interactive tutoring.

#### Completion criteria

Section 5.8 is complete when users can upload and download learning files through presigned URLs, large videos complete multipart transfer, the backend reads PDF/DOCX/scanned PDF content, tutors generate document-based summaries and quizzes, students chat within lesson context, and model/token usage is persisted.
