---
title: "Week 6 Worklog"
date: 2026-07-06
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 objectives

* Integrate an AI assistant into LearnSphere with a configurable provider architecture.
* Allow AI features to use lesson documents for chat, summarization, and question generation.
* Support OCR for scanned PDFs and safely control document indexing.
* Complete chat history management, request limits, and AI token tracking.
* Ensure S3 and multipart upload workflows do not leave unused objects.

### Tasks completed during the week

| Day | Tasks | Start date | Completion date | Reference material |
| --- | --- | --- | --- | --- |
| **2** | - Integrated Amazon Bedrock and Groq for AI features.<br>- Created an AI Provider layer that allowed model changes through environment variables without changing controllers.<br>- Added primary and fallback providers with automatic failover for throttling, timeout, and service errors.<br>- Standardized AI errors for quotas, access permissions, credentials, configuration, timeout, and service availability.<br>- Stored model IDs and input/output token counts to support cost tracking. | 07/06/2026 | 07/06/2026 | https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-call.html<br>https://console.groq.com/docs/text-chat<br>https://docs.aws.amazon.com/general/latest/gr/bedrock.html |
| **3** | - Completed the contextual AI chat API for courses and lessons.<br>- Enforced access rules so students could only use AI in active enrollments and tutors could only access their own courses.<br>- Included recent chat turns as context and stored message history in MongoDB.<br>- Added APIs for retrieving and deleting chat history.<br>- Added per-user rate limiting and clear responses for 401, 403, AI_THROTTLED, and AI_RATE_LIMITED errors. | 07/07/2026 | 07/07/2026 | https://console.groq.com/docs/rate-limits<br>https://console.groq.com/docs/errors<br>https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-call.html |
| **4** | - Implemented document indexing for learning materials stored on Amazon S3.<br>- Extracted content from PDF and DOCX files with pdf-parse and Mammoth.<br>- Stored the index status, normalized content, and document source fingerprint in each lesson.<br>- Automatically indexed unprocessed documents when tutors generated summaries or quiz questions.<br>- Prevented AI from generating unsupported content when a document could not be read. | 07/08/2026 | 07/08/2026 | https://www.npmjs.com/package/pdf-parse<br>https://github.com/mwilliamson/mammoth.js/<br>https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html |
| **5** | - Added OCR for scanned PDF documents with Vietnamese Tesseract data.<br>- Rendered and processed pages sequentially to reduce memory usage.<br>- Configured maximum pages, image width, timeout, and OCR concurrency limits.<br>- Added run IDs, source-change checks, and recovery for interrupted jobs.<br>- Prevented outdated OCR results from overwriting the index of a newer document. | 07/09/2026 | 07/09/2026 | https://github.com/naptha/tesseract.js/<br>https://github.com/naptha/tesseract.js/blob/master/docs/api.md |
| **6** | - Completed document summarization and cached results for student reuse to avoid repeated AI calls.<br>- Completed quiz question generation by count and basic, intermediate, or advanced difficulty.<br>- Validated and repaired AI-generated JSON before presenting drafts to tutors.<br>- Completed presigned and multipart upload workflows for large videos.<br>- Added upload sessions, multipart retry/abort, orphan-object cleanup, and durable retry tasks for failed S3 deletions. | 07/10/2026 | 07/10/2026 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html<br>https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html<br>https://console.groq.com/docs/text-chat |

### Week 6 achievements

* Implemented an AI Provider layer supporting Amazon Bedrock, Groq, and automatic fallback.

* Standardized AI errors so the Frontend could display appropriate messages.
* Completed contextual AI chat with retrieve and delete history APIs.
* Stored model IDs and input/output token counts for each AI response.
* Added per-user AI request limits to control costs.
* Extracted and indexed content from:
  * Text-based PDF files
  * DOCX files
  * Scanned PDFs through OCR
* Protected OCR processing with resource limits, timeouts, run IDs, and source-change checks.
* Cached document summaries for student reuse instead of calling the model on every view.
* Generated quiz drafts by difficulty and validated AI response structures.
* Completed multipart uploads, orphan-object cleanup, and reliable S3 cleanup retries.
