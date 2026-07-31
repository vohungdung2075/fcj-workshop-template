---
title: "Document extraction and OCR"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.8.3. </b> "
---

#### 1. Trigger document processing

A tutor uploads a lesson document and the backend processes it through:

```text
POST /api/lessons/:lesson_id/ai-index
```

The endpoint requires authentication, the `tutor` role, and AI rate limiting. The backend reads the S3 object by `document_key`; the browser never resends the complete file in an AI request.

#### 2. Select the extractor

| File type | Processing |
| --- | --- |
| Text PDF | `pdf-parse` / PDF parser |
| DOCX | `mammoth.extractRawText` |
| Scanned PDF | page rendering and `tesseract.js` Vietnamese OCR |

If normalized PDF text is shorter than `AI_PDF_OCR_MIN_TEXT_CHARS`, processing automatically falls back to OCR. This allows scanned materials to support summaries and quizzes.

#### 3. Resource limits

Production limits include:

```dotenv
AI_DOCUMENT_MAX_BYTES=20971520
AI_INDEX_MAX_CHARS=200000
AI_PDF_OCR_MAX_PAGES=12
AI_PDF_OCR_IMAGE_WIDTH=1400
AI_PDF_OCR_TIMEOUT_MS=120000
AI_PDF_OCR_MAX_CONCURRENT=1
```

OCR processes pages sequentially and limits concurrency to protect EC2 CPU and memory. Oversized documents should be split into smaller lessons.

#### 4. Persist index and status

Extracted text is normalized, bounded, and stored with the lesson, so chat, summary, and quiz operations do not repeat OCR. Processing state distinguishes:

```text
processing → ready
processing → failed
```

A job older than `AI_INDEX_STALE_MS` is treated as interrupted and can run again. The UI surfaces specific errors for empty text, unsupported type, OCR timeout, or OCR busy.

#### 5. Relationship to summaries

When a tutor requests a summary before indexing, the backend indexes the document first. A summary is tied to `document_key`; if the source changes during generation, the old response is not written into the new lesson.

#### Deployment evidence

![Lesson resource panel showing that the document is ready for AI](/images/learnsphere-ai-document-ready.png)

**Figure 5.51:** The tutor lesson page reports that the document has been read and is ready for AI operations. The same panel keeps the original learning document available to authorized users.

![Vietnamese learning content and chemical formulas rendered after document processing](/images/learnsphere-rendered-learning-content.png)

**Figure 5.52:** Processed learning content is rendered with readable Vietnamese text and chemical subscripts. The sample confirms that the extraction and presentation pipeline preserves meaningful instructional notation without exposing personal data.
