---
title: "Worklog Tuần 7"
date: 2026-07-13
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7

* Hoàn thiện giao diện LearnSphere theo từng vai trò student, tutor và admin.
* Kết nối đầy đủ Frontend với các API khóa học, bài học, quiz, AI và enrollment.
* Cải thiện trải nghiệm quản lý nội dung, học tập và làm bài quiz.
* Kiểm tra, sửa lỗi nghiệp vụ và chuẩn bị ứng dụng cho môi trường production.
* Chuẩn bị cấu hình và kiểm tra ứng dụng trước khi triển khai lên AWS.

### Công việc thực hiện trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| **2** | - Hoàn thiện application shell, Header, Sidebar và điều hướng theo vai trò.<br>- Bổ sung protected route, trang 404, redirect và navigation SPA.<br>- Hoàn thiện trang hồ sơ, cập nhật avatar và hiển thị toast cho loading, thành công, lỗi.<br>- Cải thiện bố cục responsive và kích thước cửa sổ Sphere AI để nội dung dễ đọc. | 13/07/2026 | 13/07/2026 | https://react.dev/learn<br>https://reactrouter.com/start/declarative/routing |
| **3** | - Hoàn thiện trang danh sách, chi tiết và quản lý khóa học.<br>- Bổ sung tìm kiếm, bộ lọc, khóa học phổ biến và carousel tự động chuyển.<br>- Cải thiện hiển thị thumbnail, số học viên active và trạng thái đăng ký.<br>- Tách trang tổng quan khóa học khỏi bài học đầu tiên.<br>- Hoàn thiện form tạo, sửa, xóa, khôi phục khóa học và giao diện quản lý bài học. | 14/07/2026 | 14/07/2026 | https://react.dev/learn<br>https://vite.dev/guide/ |
| **4** | - Hoàn thiện trang chi tiết bài học và danh sách module.<br>- Cho phép tutor tạo bài học và chuyển trực tiếp đến bài học vừa tạo.<br>- Tích hợp upload video, document, thumbnail và hiển thị tiến trình upload.<br>- Đưa tài liệu lên vị trí nổi bật và chỉ hiển thị trạng thái AI document cho tutor/admin.<br>- Tích hợp tóm tắt document, hiển thị công thức toán học và xử lý trạng thái video dung lượng lớn. | 15/07/2026 | 15/07/2026 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html<br>https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video |
| **5** | - Sắp xếp lại giao diện quản lý quiz theo quy trình chọn khóa học, chọn quiz, thêm câu hỏi và kiểm tra.<br>- Tích hợp sinh câu hỏi bằng AI theo số lượng và mức độ khó.<br>- Đưa nút thêm bản nháp AI xuống cuối danh sách để tutor kiểm tra trước khi lưu.<br>- Hoàn thiện giao diện làm quiz, timer, tự động nộp đáp án mới nhất và ngăn submit trùng.<br>- Bổ sung hiển thị công thức toán học bằng KaTeX và xử lý retry khi submit thất bại. | 16/07/2026 | 16/07/2026 | https://katex.org/docs/supported<br>https://react.dev/learn/updating-arrays-in-state |
| **6** | - Hoàn thiện giao diện enrollment cho học viên và giảng viên.<br>- Bổ sung hủy đăng ký, phê duyệt, từ chối và xóa học viên khỏi khóa học.<br>- Xây dựng cửa sổ báo cáo học tập khi tutor chọn tên học viên, gồm tiến độ và chi tiết lượt làm quiz.<br>- Điều chỉnh Dashboard theo vai trò và hoàn thiện trang quản trị người dùng, khóa học, quiz, system monitoring.<br>- Chạy kiểm tra Frontend/Backend, sửa lỗi logic và chuẩn bị cấu hình production. | 17/07/2026 | 17/07/2026 | https://react.dev/learn<br>https://www.mongodb.com/docs/manual/aggregation/ |

### Kết quả đạt được tuần 7

* Hoàn thiện điều hướng SPA, route bảo vệ và giao diện theo từng vai trò.

* Hoàn thiện trang hồ sơ và quy trình upload avatar.
* Cải thiện danh sách khóa học với tìm kiếm, bộ lọc, carousel khóa học phổ biến và số học viên thực tế.
* Hoàn thiện quy trình quản lý khóa học, bài học và học liệu.
* Tích hợp đầy đủ AI chat, tóm tắt document và sinh câu hỏi vào Frontend.
* Hoàn thiện giao diện quản lý và làm quiz, bao gồm timer và auto-submit an toàn.
* Hoàn thiện luồng enrollment cho student và tutor.
* Hiển thị được báo cáo tiến độ học và chi tiết kết quả quiz của từng học viên.
* Hoàn thiện các trang quản trị dành cho admin.
* Kiểm tra build, sửa lỗi nghiệp vụ và chuẩn bị ứng dụng cho production.
