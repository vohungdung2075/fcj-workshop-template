---
title: "Worklog Tuần 4"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4

* Hiểu cách theo dõi tài nguyên, thu thập log và tạo cảnh báo với Amazon CloudWatch.
* Biết cách sử dụng Amazon SNS để gửi thông báo khi hệ thống xảy ra sự cố.
* Tìm hiểu Amazon Bedrock và cách tích hợp mô hình nền tảng vào ứng dụng.
* Nắm được quy trình tự động kiểm thử, đóng gói và triển khai mã nguồn bằng GitHub Actions.
* Tổng hợp các dịch vụ đã học để thiết kế kiến trúc cho một ứng dụng web trên AWS.

### Công việc thực hiện trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| **2** | - Tìm hiểu tổng quan về **Amazon CloudWatch** và vai trò của dịch vụ trong giám sát hệ thống.<br>- Nghiên cứu các thành phần chính:<br>&emsp;+ Metrics<br>&emsp;+ Logs và Log Groups<br>&emsp;+ Dashboards<br>&emsp;+ Alarms<br>- Thực hành theo dõi các metric của EC2 như CPUUtilization, NetworkIn và NetworkOut.<br>- Tạo CloudWatch Dashboard để tổng hợp các thông tin giám sát cơ bản. | 22/06/2026 | 22/06/2026 | https://000008.awsstudygroup.com/ |
| **3** | - Tìm hiểu **Amazon Simple Notification Service (SNS)** và mô hình publish/subscribe.<br>- Nghiên cứu các thành phần:<br>&emsp;+ Topic<br>&emsp;+ Publisher<br>&emsp;+ Subscription<br>&emsp;+ Endpoint<br>- Thực hành tạo SNS topic, đăng ký nhận thông báo qua email và xác nhận subscription.<br>- Kết nối CloudWatch Alarm với SNS để gửi cảnh báo khi metric vượt ngưỡng cấu hình. | 23/06/2026 | 23/06/2026 | https://000077.awsstudygroup.com/|
| **4** | - Tìm hiểu tổng quan về **Amazon Bedrock** và khái niệm Foundation Model.<br>- Khám phá Model Catalog, Bedrock Playground và các hình thức inference.<br>- Tìm hiểu cách ứng dụng gửi prompt và nhận phản hồi từ mô hình thông qua API.<br>- Nghiên cứu quyền IAM cần thiết để gọi model, cách lựa chọn Region, model ID và inference profile.<br>- Tìm hiểu ảnh hưởng của token, quota và throttling đến chi phí và khả năng hoạt động của tính năng AI. | 24/06/2026 | 24/06/2026 | https://000056.awsstudygroup.com/ |
| **5** | - Tìm hiểu quy trình **Continuous Integration và Continuous Deployment (CI/CD)** trong phát triển phần mềm.<br>- Làm quen với GitHub Actions và cấu trúc workflow YAML:<br>&emsp;+ Trigger<br>&emsp;+ Job<br>&emsp;+ Step<br>&emsp;+ Secret và Variable<br>- Tìm hiểu quy trình build, kiểm thử và đóng gói ứng dụng thành Docker image.<br>- Nghiên cứu cách GitHub Actions xác thực với AWS bằng OpenID Connect (OIDC) thay cho việc lưu Access Key dài hạn. | 25/06/2026 | 25/06/2026 | https://000051.awsstudygroup.com/ |
| **6** | - Tổng hợp các dịch vụ AWS đã học trong bốn tuần đầu.<br>- Thiết kế kiến trúc mẫu cho một ứng dụng web gồm:<br>&emsp;+ Amazon S3 và CloudFront phân phối Frontend<br>&emsp;+ Amazon EC2 chạy Backend bằng Docker<br>&emsp;+ Amazon ECR lưu trữ Docker image<br>&emsp;+ Amazon CloudWatch và SNS giám sát, cảnh báo<br>&emsp;+ IAM và VPC kiểm soát quyền truy cập, kết nối mạng<br>- Xem xét kiến trúc theo các khía cạnh bảo mật, hiệu năng, độ tin cậy và chi phí.<br>- Chuẩn bị kế hoạch chuyển sang giai đoạn phát triển dự án LearnSphere. | 26/06/2026 | 26/06/2026 | https://cloudjourney.awsstudygroup.com/ |

### Kết quả đạt được tuần 4

* Hiểu các thành phần giám sát chính của Amazon CloudWatch:
  * Metrics
  * Logs
  * Dashboards
  * Alarms

* Theo dõi được các metric cơ bản của EC2 và tạo CloudWatch Dashboard.
* Tạo được SNS topic, email subscription và kết nối SNS với CloudWatch Alarm.
* Hiểu vai trò của Amazon Bedrock trong việc tích hợp AI tạo sinh vào ứng dụng.
* Nắm được các yếu tố cần thiết khi gọi model:
  * Region
  * Model ID
  * Inference Profile
  * IAM Permission
  * Token và Quota
* Hiểu cấu trúc cơ bản của GitHub Actions workflow gồm Trigger, Job và Step.
* Nắm được quy trình build, kiểm thử, tạo Docker image và triển khai ứng dụng tự động.
* Hiểu lợi ích của việc sử dụng OIDC để GitHub Actions truy cập AWS mà không lưu Access Key dài hạn.
* Hoàn thành sơ đồ kiến trúc mẫu kết hợp S3, CloudFront, EC2, ECR, CloudWatch, SNS, IAM và VPC.
* Hoàn thành giai đoạn học tập AWS nền tảng và chuẩn bị chuyển sang phát triển dự án LearnSphere.
