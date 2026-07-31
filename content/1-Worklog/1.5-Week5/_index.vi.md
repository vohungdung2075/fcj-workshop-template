---
title: "Worklog Tuần 5"
date: 2026-06-29
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5

* Phân tích kiến trúc, trạng thái bàn giao và yêu cầu nghiệp vụ của dự án LearnSphere.
* Hoàn thiện các chức năng Backend cốt lõi cho tài khoản, khóa học, bài học và quiz.
* Xây dựng luồng đăng ký khóa học, theo dõi tiến độ và thông báo theo từng vai trò.
* Bảo đảm dữ liệu MongoDB và tệp học liệu trên Amazon S3 được quản lý an toàn.

### Công việc thực hiện trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| **2** | - Đọc tài liệu bàn giao, API Design và kiểm tra cấu trúc thực tế của LearnSphere.<br>- Phân tích kiến trúc Frontend React/Vite, Backend Express.js và cơ sở dữ liệu MongoDB.<br>- Kiểm tra các model, route, controller, service và biến môi trường hiện có.<br>- Xác định các chức năng đã hoàn thành, chức năng còn thiếu và lập kế hoạch phát triển cho bốn tuần cuối. | 29/06/2026 | 29/06/2026 | https://react.dev/learn/creating-a-react-app<br>https://expressjs.com/en/guide/routing.html<br>https://www.mongodb.com/docs/manual/contents/ |
| **3** | - Hoàn thiện luồng xác thực và quản lý tài khoản:<br>&emsp;+ Đăng ký và đăng nhập<br>&emsp;+ JWT và cookie xác thực<br>&emsp;+ Quên và đặt lại mật khẩu<br>&emsp;+ Phân quyền student, tutor, admin<br>&emsp;+ Trạng thái pending, active, blocked<br>- Bổ sung API cập nhật hồ sơ cá nhân và avatar.<br>- Kiểm tra middleware xác thực, phân quyền và phản hồi lỗi 401/403. | 30/06/2026 | 30/06/2026 | https://expressjs.com/en/advanced/best-practice-security.html<br>https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies |
| **4** | - Hoàn thiện các API quản lý khóa học và bài học.<br>- Xây dựng luồng tạo, xem, cập nhật, soft-delete, khôi phục và xóa vĩnh viễn khóa học.<br>- Hoàn thiện CRUD bài học, sắp xếp thứ tự và ghi nhận tiến độ hoàn thành.<br>- Lưu video, document và thumbnail bằng S3 object key thay vì lưu URL tạm thời.<br>- Bổ sung cơ chế xóa file S3 an toàn khi thay thế hoặc xóa dữ liệu. | 01/07/2026 | 01/07/2026 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html<br>https://www.mongodb.com/docs/manual/core/transactions/ |
| **5** | - Hoàn thiện nghiệp vụ quiz và câu hỏi:<br>&emsp;+ Tạo, sửa, xóa quiz<br>&emsp;+ Quản lý câu hỏi và đáp án<br>&emsp;+ Bắt đầu lượt làm bài<br>&emsp;+ Giới hạn thời gian và tự động hết hạn<br>&emsp;+ Nộp bài, chấm điểm và lưu lịch sử<br>- Lưu snapshot câu hỏi tại thời điểm bắt đầu để kết quả không bị ảnh hưởng khi quiz được chỉnh sửa.<br>- Ngăn chỉnh sửa quiz khi học viên đang có lượt làm bài hợp lệ. | 02/07/2026 | 02/07/2026 | https://www.mongodb.com/docs/manual/core/transactions/<br>https://mongoosejs.com/docs/validation.html |
| **6** | - Hoàn thiện luồng đăng ký khóa học mở và khóa học cần phê duyệt.<br>- Bổ sung chức năng học viên đăng ký, hủy đăng ký; giảng viên phê duyệt, từ chối hoặc xóa học viên.<br>- Bảo toàn yêu cầu pending khi giảng viên chuyển khóa học sang hình thức đăng ký mở.<br>- Hoàn thiện API khóa học của tôi, tiến độ khóa học và báo cáo kết quả quiz.<br>- Kiểm tra hệ thống thông báo và thảo luận trong khóa học.<br>- Cập nhật tài liệu API theo các chức năng đã hoàn thiện. | 03/07/2026 | 03/07/2026 | https://www.mongodb.com/docs/manual/data-modeling/<br>https://expressjs.com/en/guide/routing.html |

### Kết quả đạt được tuần 5

* Hiểu rõ kiến trúc và trạng thái bàn giao của dự án LearnSphere.

* Hoàn thiện luồng xác thực, cập nhật hồ sơ và phân quyền:
  * Student
  * Tutor
  * Admin
* Hoàn thiện vòng đời khóa học gồm tạo, cập nhật, soft-delete, khôi phục và xóa vĩnh viễn.
* Hoàn thiện CRUD bài học, quản lý học liệu S3 và ghi nhận tiến độ học tập.
* Hoàn thiện luồng làm quiz có giới hạn thời gian, snapshot câu hỏi, chấm điểm và lưu lịch sử.
* Xây dựng đầy đủ nghiệp vụ đăng ký, hủy đăng ký, phê duyệt và quản lý học viên trong khóa học.
* Bảo đảm luồng thay đổi loại đăng ký không làm mất yêu cầu pending của học viên.
* Hoàn thiện API xem tiến độ khóa học, kết quả quiz, thông báo và thảo luận.
* Đồng bộ tài liệu API với mã nguồn Backend.
