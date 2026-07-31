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

#### Tiêu chí hoàn thành

Mục 5.8 hoàn tất khi người dùng upload/download học liệu qua presigned URL, video lớn hoàn tất multipart, Backend đọc được PDF/DOCX/PDF scan, tutor tạo summary/quiz từ tài liệu, học viên chat theo bài học và hệ thống lưu model/token cùng lịch sử phù hợp.
