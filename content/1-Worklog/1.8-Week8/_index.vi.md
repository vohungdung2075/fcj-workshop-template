---
title: "Worklog Tuần 8"
date: 2026-07-20
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8

* Hoàn thiện hardening, kiểm thử và cấu hình production cho LearnSphere.
* Triển khai Backend, Frontend và media storage lên các dịch vụ AWS.
* Xây dựng quy trình CI/CD tự động bằng GitHub Actions và AWS OIDC.
* Nghiệm thu các chức năng trên môi trường production.
* Thiết lập giám sát, cảnh báo và hoàn thiện sơ đồ kiến trúc hệ thống.

### Công việc thực hiện trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| **2** | - Kiểm tra toàn bộ project trước khi triển khai.<br>- Bổ sung validation biến môi trường và cấu hình production.<br>- Cấu hình HttpOnly/Secure cookie, Helmet, CORS allowlist, rate limit và giới hạn JSON request.<br>- Hoàn thiện 404 handler, global error handler, graceful shutdown và health check live/ready.<br>- Sửa cảnh báo Mongoose, loại bỏ file thừa và chạy Backend test cùng Frontend production build. | 20/07/2026 | 20/07/2026 | https://expressjs.com/en/advanced/best-practice-security.html<br>https://docs.docker.com/guides/nodejs/ |
| **3** | - Chuẩn bị hạ tầng AWS cho LearnSphere.<br>- Tạo private S3 bucket cho Frontend và media; cấu hình CloudFront Origin Access Control.<br>- Tạo Amazon ECR repository và EC2 instance chạy Docker.<br>- Gắn IAM Role cho EC2 để truy cập ECR, S3, CloudWatch và Bedrock theo quyền cần thiết.<br>- Cấu hình MongoDB Atlas, Security Group, biến môi trường và kiểm tra Docker trên EC2. | 21/07/2026 | 21/07/2026 | https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html<br>https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html<br>https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html |
| **4** | - Hoàn thiện GitHub Actions workflow triển khai tự động.<br>- Cấu hình GitHub OIDC và IAM Role cho phép AssumeRoleWithWebIdentity.<br>- Build, test, tag Docker image theo commit SHA và push lên Amazon ECR.<br>- Tự động cập nhật container Backend trên EC2 và kiểm tra health endpoint.<br>- Build Frontend, đồng bộ file lên S3 và invalidate CloudFront cache. | 22/07/2026 | 22/07/2026 | https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws<br>https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html |
| **5** | - Nghiệm thu hệ thống LearnSphere trên domain CloudFront HTTPS.<br>- Kiểm tra đăng ký, đăng nhập, phân quyền, quản lý khóa học, bài học và enrollment.<br>- Kiểm tra upload thumbnail, avatar, document và video multipart qua presigned URL.<br>- Kiểm tra xem video, tóm tắt document, chat AI, sinh quiz, làm bài và xem kết quả.<br>- Sửa lỗi OIDC, CORS, AI response structure và xác nhận container Backend ở trạng thái healthy. | 23/07/2026 | 23/07/2026 | https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html<br>https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html |
| **6** | - Hoàn thiện API thống kê và trang System Monitoring cho admin.<br>- Thiết lập Amazon CloudWatch Alarm cho CPUUtilization và StatusCheckFailed.<br>- Tạo Amazon SNS topic và email subscription nhận cảnh báo.<br>- Vẽ sơ đồ kiến trúc hệ thống và mô tả luồng giữa CloudFront, S3, EC2, ECR, IAM, CloudWatch, SNS, MongoDB Atlas và AI Provider. | 24/07/2026 | 24/07/2026 | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating_status_check_alarms.html<br>https://docs.aws.amazon.com/sns/latest/dg/welcome.html<br>https://aws.amazon.com/architecture/icons/ |

### Kết quả đạt được tuần 8

* Hoàn thiện kiểm tra bảo mật, validation cấu hình, error handling và health check cho Backend.

* Backend test và Frontend production build hoàn thành thành công.
* Triển khai Backend bằng Docker trên EC2 với image lưu tại Amazon ECR.
* Triển khai Frontend lên private S3 bucket và phân phối qua Amazon CloudFront HTTPS.
* Cấu hình IAM Role cho EC2 và GitHub Actions theo nguyên tắc không lưu AWS Access Key dài hạn.
* Hoàn thiện GitHub Actions CI/CD cho Backend và Frontend.
* Nghiệm thu các chức năng chính trên môi trường production.
* Xác nhận container Backend hoạt động healthy và kết nối MongoDB thành công.
* Hoàn thiện trang System Monitoring cùng các metric hệ thống và dung lượng S3.
* Thiết lập CloudWatch Alarm và nhận cảnh báo qua Amazon SNS.
* Hoàn thiện sơ đồ kiến trúc và mô tả luồng hoạt động giữa các thành phần của hệ thống LearnSphere.
