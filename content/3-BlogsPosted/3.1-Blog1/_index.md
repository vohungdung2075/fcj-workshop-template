---
title: "Blog 1"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Why Our Team Didn't Use Amazon Bedrock Knowledge Bases for Our RAG Chatbot

Hello everyone,

While building a document Q&A chatbot using Retrieval-Augmented Generation (RAG), our team used Amazon Bedrock to invoke Anthropic Claude for answering user questions based on uploaded documents. During our research on building RAG on AWS, we noticed that almost every guide mentions **Knowledge Bases for Amazon Bedrock**.

This makes total sense. Knowledge Bases handles almost the entire RAG pipeline out-of-the-box:
- Document parsing
- Text chunking
- Embedding creation
- Vector storage
- Context retrieval

It sounds extremely appealing. Initially, I thought: *"Why build anything custom at all?"* But after reading the AWS Blog and evaluating our project's specific requirements, our team decided **not to use Knowledge Bases and instead build our custom pipeline with FAISS**.

---

### What Does Knowledge Bases Do?

AWS describes Knowledge Bases as a **Fully Managed RAG** service. By pointing to a document storage source (such as Amazon S3), Bedrock automatically handles:
1. Ingesting documents from S3.
2. Chunking text content.
3. Generating vector embeddings.
4. Storing vectors in a managed Vector Database.
5. Retrieving relevant context when queried.
6. Passing context to the Foundation Model (LLM).

In short, many steps that developers usually write from scratch are completely automated by AWS.

---

### Why We Initially Planned to Use It

When first reading the documentation, it seemed like the perfect solution:
- No need to manage FAISS or custom vector stores.
- No need to write custom ingestion pipelines.
- No complex vector database management.
- Everything works out-of-the-box.

---

### Re-evaluating Our Project Requirements...

Here is what triggered our decision shift. Our chatbot doesn't just receive standard text PDFs. Users also upload:
- Scanned PDF documents
- DOCX files
- Documents with complex tables
- Documents containing embedded images

Some files require **OCR preprocessing**, some require specialized parsing, some need **heading-based chunking**, while others need **section-based chunking**. With Knowledge Bases, the ingestion pipeline is fully managed by AWS, whereas our team needed **full end-to-end control over document ingestion**.

---

### Why We Chose a Custom Pipeline with FAISS

Although using FAISS requires more custom development work, it gives us full autonomy to:
- **Execute custom OCR with Amazon Textract** for scanned files.
- **Tailor chunking strategies** per document type.
- **Attach custom metadata** per user and document category.
- **Implement multi-tenant data isolation** seamlessly.
- **Update the vector store instantly** when users delete documents.

These features were essential for our application architecture.

---

### Key Takeaways

Reading the AWS Blog led to an interesting realization: **Knowledge Bases is not a "superior version" of FAISS**. It is simply a different approach to building RAG:

- For **rapid deployment and low operational overhead**, Knowledge Bases is an ideal choice.
- For **deep customization and complete pipeline control**, building custom RAG with FAISS remains highly advantageous.

There is no single "correct" answer—what matters most is choosing the architecture that best fits your project's specific demands.

---

### Conclusion

Initially, I assumed that since AWS provides Knowledge Bases, custom RAG pipelines were obsolete. After deeper technical evaluation, I realized Knowledge Bases excels at rapid, fully-managed deployments. Meanwhile, projects requiring custom OCR, advanced chunking, rich metadata, or multi-tenancy still have strong reasons to build custom pipelines.

For our chatbot, using FAISS wasn't about being "better" than Knowledge Bases—it was simply **better suited to how our system was designed**.

---

### REFERENCE BLOG LINK

- **AWS News Blog** – *Knowledge Bases now delivers fully managed RAG experience in Amazon Bedrock*:  
  [https://aws.amazon.com/blogs/aws/knowledge-bases-now-delivers-fully-managed-rag-experience-in-amazon-bedrock/](https://aws.amazon.com/blogs/aws/knowledge-bases-now-delivers-fully-managed-rag-experience-in-amazon-bedrock/)