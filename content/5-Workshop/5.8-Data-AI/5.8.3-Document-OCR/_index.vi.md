---
title: "Trích xuất tài liệu và OCR"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.8.3. </b> "
---

#### 1. Kích hoạt xử lý tài liệu

Tutor tải tài liệu lên bài học rồi Backend xử lý:

```text
POST /api/lessons/:lesson_id/ai-index
```

Endpoint yêu cầu xác thực, role `tutor` và AI rate limit. Backend đọc object từ S3 theo `document_key`; trình duyệt không gửi lại toàn bộ file qua request AI.

#### 2. Chọn bộ trích xuất

| Loại file | Xử lý |
| --- | --- |
| PDF có text | `pdf-parse` / PDF parser |
| DOCX | `mammoth.extractRawText` |
| PDF scan | render từng trang và OCR bằng `tesseract.js` + tiếng Việt |

Nếu text PDF sau chuẩn hóa ngắn hơn `AI_PDF_OCR_MIN_TEXT_CHARS`, hệ thống tự chuyển sang OCR. Đây là lý do tài liệu scan vẫn có thể dùng cho summary và quiz.

#### 3. Giới hạn tài nguyên

Các giới hạn production:

```dotenv
AI_DOCUMENT_MAX_BYTES=20971520
AI_INDEX_MAX_CHARS=200000
AI_PDF_OCR_MAX_PAGES=12
AI_PDF_OCR_IMAGE_WIDTH=1400
AI_PDF_OCR_TIMEOUT_MS=120000
AI_PDF_OCR_MAX_CONCURRENT=1
```

OCR xử lý từng trang và giới hạn concurrency để tránh làm cạn RAM/CPU trên EC2. Tài liệu quá lớn cần chia thành bài học nhỏ hơn.

#### 4. Lưu index và trạng thái

Text được chuẩn hóa, giới hạn ký tự và lưu cùng lesson để các lần chat/summary/quiz không phải OCR lại. Trạng thái xử lý phân biệt:

```text
processing → ready
processing → failed
```

Một job cũ quá `AI_INDEX_STALE_MS` được xem là gián đoạn và có thể chạy lại. Frontend hiển thị lỗi cụ thể cho tài liệu rỗng, loại không hỗ trợ, OCR timeout hoặc OCR đang bận.

#### 5. Quan hệ với summary

Khi tutor yêu cầu summary mà tài liệu chưa có index, Backend tự index trước. Summary được gắn với `document_key`; nếu tài liệu thay đổi trong lúc sinh, kết quả cũ không được ghi đè vào lesson mới.

#### Bằng chứng triển khai

![Khu vực tài nguyên bài học hiển thị document đã sẵn sàng cho AI](/images/learnsphere-ai-document-ready.png)

**Hình 5.51:** Trang bài học của tutor xác nhận document đã được đọc và sẵn sàng cho các tác vụ AI. Cùng khu vực này vẫn cung cấp tài liệu gốc cho người dùng có quyền truy cập.

![Nội dung tiếng Việt và công thức hóa học được hiển thị sau khi xử lý document](/images/learnsphere-rendered-learning-content.png)

**Hình 5.52:** Nội dung học tập sau xử lý hiển thị rõ tiếng Việt và chỉ số dưới trong công thức hóa học. Tài liệu mẫu chứng minh pipeline trích xuất và trình bày giữ được ký hiệu giảng dạy quan trọng mà không chứa dữ liệu cá nhân.
