---
title: "Groq AI, context và kiểm soát usage"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.8.4. </b> "
---

#### 1. Provider production

Production sử dụng:

```dotenv
AI_PROVIDER=groq
GROQ_MODEL=llama-3.3-70b-versatile
AI_PROVIDER_TIMEOUT_MS=120000
```

`GROQ_API_KEY` chỉ nằm trong SSM SecureString và được EC2 đọc lúc bootstrap. Frontend không nhận API key và không gọi Groq trực tiếp.

#### 2. Các API AI

| API | Quyền | Chức năng |
| --- | --- | --- |
| `POST /api/ai/chat` | user đã đăng nhập | Hỏi đáp theo lesson context |
| `GET /api/ai/history` | user đã đăng nhập | Lấy lịch sử chat |
| `DELETE /api/ai/history` | user đã đăng nhập | Xóa lịch sử chat |
| `POST /api/ai/summarize-lesson/:lesson_id` | theo course access | Lấy/tạo summary tài liệu |
| `POST /api/ai/generate-quiz` | tutor | Sinh câu hỏi theo lesson và difficulty |

Student chỉ dùng context khóa học có enrollment active. Tutor có thể index, tạo lại summary và sinh quiz; Backend không tin `lesson_id` từ client nếu course relationship không hợp lệ.

#### 3. Giới hạn context

```dotenv
AI_CHAT_CONTEXT_MAX_CHARS=7000
AI_SUMMARY_CONTEXT_MAX_CHARS=9000
AI_QUIZ_CONTEXT_MAX_CHARS=8000
```

Backend chọn đoạn liên quan từ indexed text thay vì gửi toàn bộ document. Chat dùng thêm một số message gần nhất; quiz prompt nhận difficulty để phân biệt cơ bản, trung bình và nâng cao.

#### 4. Summary cache và quiz validation

Summary được lưu với:

* `document_key`, model ID và thời gian sinh.
* Input/output token và stop reason.
* Trạng thái `processing`, `ready` hoặc `failed`.

Học viên đọc bản summary đã lưu; tutor mới có quyền force regenerate. Với quiz, response AI phải qua structured parser, đúng số câu, đúng option và đáp án Boolean/JSON hợp lệ trước khi tạo bản nháp. Tutor vẫn kiểm tra trước khi lưu vào quiz.

#### 5. Rate limit và xử lý lỗi

```dotenv
AI_RATE_LIMIT_REQUESTS=10
AI_RATE_LIMIT_WINDOW_MS=60000
```

Rate limit được tính theo user và lưu bền vững trong MongoDB, do đó không bị nhân đôi khi request đi qua hai EC2. Khi vượt giới hạn, API trả `429 AI_RATE_LIMITED` cùng `Retry-After`.

Lỗi provider được chuẩn hóa:

| Mã | HTTP | Ý nghĩa |
| --- | --- | --- |
| `AI_THROTTLED` | 429 | Groq vượt quota/rate |
| `AI_TIMEOUT` | 504 | Provider quá thời gian |
| `AI_CREDENTIALS_ERROR` | 503 | API key thiếu hoặc sai |
| `AI_SERVICE_UNAVAILABLE` | 503 | Provider tạm thời không sẵn sàng |
| `AI_INVALID_STRUCTURED_RESPONSE` | 502 | Quiz trả cấu trúc không hợp lệ |

#### 6. Theo dõi model và token

`AIMessage` lưu `model_id`, `input_tokens`, `output_tokens`, tổng token và stop reason. Giao diện AI Assistant hiển thị model/token; dữ liệu này hỗ trợ ước tính chi phí, theo dõi usage và điều tra response bất thường.

Không ghi prompt đầy đủ, API key hoặc dữ liệu nhạy cảm vào log CloudWatch.

#### Bằng chứng triển khai

![Sphere AI Assistant trả lời trong ngữ cảnh khóa học và bài học đã chọn](/images/learnsphere-contextual-ai-assistant.png)

**Hình 5.53:** Sphere AI trả lời câu hỏi khi giao diện xác định rõ khóa học và bài học đang được dùng làm ngữ cảnh. Ảnh chứng minh trợ lý hỗ trợ theo nội dung học tập thay vì hoạt động như một cửa sổ chat tổng quát không liên quan.

![Tóm tắt document đã lưu kèm thời gian sinh và số token](/images/learnsphere-saved-ai-summary.png)

**Hình 5.54:** Bản tóm tắt Sphere AI hiển thị trạng thái đã lưu, thời gian sinh và tổng số token. Tutor có thể sử dụng lại kết quả hoặc chủ động tạo lại, tránh phát sinh một request inference mới sau mỗi lần mở trang.

![Năm câu hỏi mức trung bình do AI tạo để tutor kiểm duyệt](/images/learnsphere-ai-generated-quiz.png)

**Hình 5.55:** Quiz builder xác nhận AI đã tạo năm câu hỏi mức trung bình và yêu cầu tutor kiểm tra trước khi sử dụng. Metadata của quiz đồng thời hiển thị thời lượng, độ khó và số câu đã lưu.
