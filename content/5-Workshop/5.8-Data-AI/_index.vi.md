---
title: "Dữ liệu, lưu trữ Media và AI"
date: 2026-07-31
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

#### Mục tiêu

Mục 5.8 mô tả luồng dữ liệu ở tầng ứng dụng sau khi hạ tầng đã được triển khai: dữ liệu nghiệp vụ dùng chung trong MongoDB Atlas, file học liệu lưu trên S3, tài liệu được trích xuất văn bản/OCR và Groq xử lý các yêu cầu AI.

Các mục tiêu chính:

* Mọi EC2 Backend trong ASG sử dụng chung một database bền vững.
* File lớn không đi xuyên qua memory hoặc filesystem của EC2.
* Upload dùng presigned URL, multipart và session tracking.
* PDF, DOCX và PDF scan được chuyển thành knowledge text có giới hạn.
* AI chat, summary và quiz dùng đúng context của khóa học.
* Token/model được lưu để theo dõi usage.
* Rate limit và mã lỗi rõ ràng bảo vệ quota.

#### Phân loại dữ liệu

| Dữ liệu | Nơi lưu | Ví dụ |
| --- | --- | --- |
| Nghiệp vụ | MongoDB Atlas | user, course, lesson, enrollment, quiz, attempt |
| Media/học liệu | `learnsphere-media-2` | thumbnail, avatar, video, PDF, DOCX |
| Cấu hình runtime | SSM Parameter Store | Backend environment, image tag |
| AI-derived data | MongoDB Atlas | indexed text, summary, history, token usage |

#### Các luồng chi tiết

1. [MongoDB Atlas và mô hình dữ liệu dùng chung](5.8.1-mongodb-data/)
2. [Presigned URL, multipart upload và dọn orphan](5.8.2-s3-media-upload/)
3. [Trích xuất tài liệu và OCR](5.8.3-document-ocr/)
4. [Groq AI, context, rate limit và usage](5.8.4-groq-ai/)

#### Ranh giới dữ liệu

```text
Metadata và trạng thái nghiệp vụ → MongoDB Atlas
Binary file                    → S3 Media
Text đã trích xuất             → Lesson AI index
Prompt + context chọn lọc      → Groq API
Response + token usage         → MongoDB Atlas
```

Không gửi toàn bộ video lên LLM. AI chỉ sử dụng text đã được trích xuất từ document và context bài học đã giới hạn để giảm latency, chi phí và rủi ro vượt context.

#### Định hướng phát triển & Tối ưu tương lai (Future Roadmap)

Để chuyển dịch LearnSphere thành một nền tảng chuẩn **AWS Native** và tối ưu toàn diện hiệu năng/bảo mật, nhóm lên kế hoạch nâng cấp 2 thành phần nòng cốt trong các phiên bản tiếp theo:

1. **Chuyển đổi Database: Từ MongoDB Atlas sang Amazon DynamoDB (AWS Native NoSQL)**
   - **Lý do nâng cấp**: Loại bỏ hoàn toàn luồng kết nối cross-cloud ra bên ngoài AWS (giảm độ trễ kết nối), tận dụng dịch vụ NoSQL Fully Managed của AWS với khả năng tự động mở rộng (Auto-scaling) và tính sẵn sàng cao (High Availability).
   - **Lợi ích**: Tích hợp sâu với **AWS IAM Policy**, **AWS KMS (Encryption at rest)**, **CloudWatch Metrics** và **VPC Gateway Endpoints** (truy cập nội bộ miễn phí không qua Internet).
   - **Giải pháp chuyển đổi**: Áp dụng thư viện ODM **`Dynamoose`** để migrate các Mongoose Schema (`User`, `Course`, `Lesson`, `QuizAttempt`...) sang DynamoDB Tables một cách nhanh chóng mà không cần thay đổi quá nhiều logic trong backend controller.

2. **Chuyển đổi AI Engine: Từ Groq API sang Amazon Bedrock (Fully Managed Generative AI trên AWS)**
   - **Lý do nâng cấp**: Đưa toàn bộ quy trình xử lý AI về hệ sinh thái AWS, tuân thủ nghiêm ngặt các tiêu chuẩn bảo mật dữ liệu doanh nghiệp và không lo rò rỉ dữ liệu ra bên thứ ba.
   - **Lợi ích**: Dễ dàng truy cập các Foundation Models tiên tiến (Anthropic Claude 3.5 Sonnet, Amazon Titan, Llama 3) qua một unified API duy nhất; tích hợp sẵn **Amazon Bedrock Guardrails** (bảo vệ nội dung), **Bedrock Knowledge Bases** (RAG tự động) và **Bedrock Agents** cho các tính năng học tập thông minh nâng cao.

#### Tiêu chí hoàn thành

Mục 5.8 hoàn tất khi người dùng upload/download học liệu qua presigned URL, video lớn hoàn tất multipart, Backend đọc được PDF/DOCX/PDF scan, tutor tạo summary/quiz từ tài liệu, học viên chat theo bài học và hệ thống lưu model/token cùng lịch sử phù hợp.
