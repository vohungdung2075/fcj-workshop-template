---
title: "Worklog Tuần 6"
date: 2026-07-06
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6

* Tích hợp trợ lý AI vào LearnSphere với kiến trúc provider có thể thay đổi.
* Cho phép AI sử dụng nội dung document của bài học để chat, tóm tắt và sinh câu hỏi.
* Hỗ trợ OCR đối với PDF scan và kiểm soát an toàn quá trình lập chỉ mục tài liệu.
* Hoàn thiện quản lý lịch sử, giới hạn request và theo dõi token AI.
* Bảo đảm các luồng upload S3 và multipart không tạo file rác.

### Công việc thực hiện trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| **2** | - Tích hợp Amazon Bedrock và Groq cho chức năng AI.<br>- Xây dựng lớp AI Provider để có thể chuyển model bằng biến môi trường mà không thay đổi controller.<br>- Bổ sung provider chính, provider dự phòng và cơ chế tự động chuyển khi gặp throttling, timeout hoặc lỗi dịch vụ.<br>- Chuẩn hóa lỗi AI gồm quota, quyền truy cập, credentials, cấu hình, timeout và service unavailable.<br>- Lưu model ID, số input token và output token để hỗ trợ theo dõi chi phí. | 06/07/2026 | 06/07/2026 | https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-call.html<br>https://console.groq.com/docs/text-chat<br>https://docs.aws.amazon.com/general/latest/gr/bedrock.html |
| **3** | - Hoàn thiện API chat AI theo ngữ cảnh khóa học và bài học.<br>- Kiểm tra quyền truy cập để học viên chỉ sử dụng AI trong khóa học đã đăng ký active và tutor chỉ truy cập khóa học sở hữu.<br>- Gửi các lượt chat gần nhất làm ngữ cảnh và lưu lịch sử vào MongoDB.<br>- Bổ sung API lấy và xóa lịch sử chat.<br>- Thêm rate limit theo user và phản hồi rõ các lỗi 401, 403, AI_THROTTLED, AI_RATE_LIMITED. | 07/07/2026 | 07/07/2026 | https://console.groq.com/docs/rate-limits<br>https://console.groq.com/docs/errors<br>https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-call.html |
| **4** | - Xây dựng quá trình lập chỉ mục document được lưu trên Amazon S3.<br>- Trích xuất nội dung từ PDF và DOCX bằng pdf-parse và Mammoth.<br>- Lưu trạng thái index, nội dung đã chuẩn hóa và dấu vết nguồn document vào bài học.<br>- Tự động lập chỉ mục khi tutor sinh câu hỏi hoặc tóm tắt từ document chưa xử lý.<br>- Ngăn AI tạo nội dung không dựa trên tài liệu khi document không thể đọc được. | 08/07/2026 | 08/07/2026 | https://www.npmjs.com/package/pdf-parse<br>https://github.com/mwilliamson/mammoth.js/<br>https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html |
| **5** | - Bổ sung OCR cho PDF scan bằng Tesseract với ngôn ngữ tiếng Việt.<br>- Render và xử lý lần lượt từng trang để giảm sử dụng bộ nhớ.<br>- Cấu hình số trang tối đa, chiều rộng ảnh, timeout và số tác vụ OCR chạy đồng thời.<br>- Bổ sung run ID, kiểm tra document thay đổi và cơ chế chạy lại job bị gián đoạn.<br>- Ngăn kết quả OCR cũ ghi đè chỉ mục của document mới. | 09/07/2026 | 09/07/2026 | https://github.com/naptha/tesseract.js/<br>https://github.com/naptha/tesseract.js/blob/master/docs/api.md |
| **6** | - Hoàn thiện chức năng tóm tắt document và lưu kết quả để học viên sử dụng lại, tránh gọi AI lặp lại.<br>- Hoàn thiện sinh câu hỏi quiz theo số lượng và mức độ cơ bản, trung bình, nâng cao.<br>- Kiểm tra và sửa cấu trúc JSON do AI trả về trước khi hiển thị bản nháp cho tutor.<br>- Hoàn thiện upload presigned URL và multipart cho video dung lượng lớn.<br>- Bổ sung UploadSession, retry/abort multipart, dọn orphan object và hàng đợi retry khi xóa S3 thất bại. | 10/07/2026 | 10/07/2026 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html<br>https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html<br>https://console.groq.com/docs/text-chat |

### Kết quả đạt được tuần 6

* Xây dựng lớp AI Provider hỗ trợ Amazon Bedrock, Groq và cơ chế fallback tự động.

* Chuẩn hóa các lỗi AI để Frontend hiển thị thông báo phù hợp.
* Hoàn thiện chat AI theo ngữ cảnh bài học cùng API lấy và xóa lịch sử.
* Lưu model ID và số input/output token của từng phản hồi AI.
* Thêm giới hạn request AI theo từng user để kiểm soát chi phí.
* Trích xuất và lập chỉ mục được nội dung:
  * PDF có lớp văn bản
  * DOCX
  * PDF scan thông qua OCR
* Bảo vệ quá trình OCR bằng giới hạn tài nguyên, timeout, run ID và kiểm tra thay đổi nguồn.
* Lưu tóm tắt document để học viên tái sử dụng thay vì gọi model ở mỗi lượt xem.
* Sinh bản nháp câu hỏi theo mức độ khó và kiểm tra cấu trúc phản hồi AI.
* Hoàn thiện upload multipart, cleanup orphan object và retry S3 cleanup an toàn.
