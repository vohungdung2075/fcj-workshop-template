---
title: "Tổng quan dự án"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Giới thiệu LearnSphere

LearnSphere là nền tảng học tập trực tuyến dành cho học viên, giảng viên và quản trị viên. Hệ thống tập trung khóa học, bài học, video, tài liệu, quiz, tiến độ học tập và tương tác AI trong cùng một ứng dụng.

#### Bối cảnh và vấn đề cần giải quyết

Các hệ thống học trực tuyến thông thường thường tách rời nội dung, bài kiểm tra và quá trình theo dõi học tập. Giảng viên phải đọc tài liệu, soạn câu hỏi và kiểm tra kết quả bằng nhiều thao tác thủ công; học viên cũng khó nhận được hỗ trợ ngay khi tự học. Video và document dung lượng lớn nếu truyền qua Backend còn làm tăng tải server và khiến việc mở rộng hệ thống phức tạp.

LearnSphere giải quyết các vấn đề trên bằng cách:

* Tập trung quản lý khóa học, bài học, học viên, quiz và tiến độ.
* Lưu media trên Amazon S3 và truyền file trực tiếp bằng presigned URL.
* Dùng nội dung document làm ngữ cảnh cho AI Assistant, tóm tắt và sinh quiz.
* Tách Frontend, Backend, dữ liệu và media để từng thành phần có thể vận hành độc lập.
* Tự động hóa triển khai production bằng GitHub Actions và AWS.

#### Mã nguồn dự án

![Repository GitHub của LearnSphere](/images/learnsphere-github-repository.png)

*Hình 5.1. Repository LearnSphere gồm Frontend, Backend, workflow CI/CD và tài liệu triển khai.*

Mã nguồn production được quản lý tại [HoiaeKHMT/LearnSphere](https://github.com/HoiaeKHMT/LearnSphere). Nhánh `main` là nguồn phát hành production; mỗi commit được GitHub Actions kiểm tra, đóng gói và triển khai theo workflow đã cấu hình.

#### Phạm vi workshop

| Nội dung | Trong phạm vi |
| --- | --- |
| Ứng dụng | Frontend React/Vite và Backend Express/Docker |
| Hạ tầng | VPC Multi-AZ, ALB, ASG, EC2 private, NAT Gateway |
| Lưu trữ | S3 Frontend, S3 Media và ECR |
| Phân phối | CloudFront, HTTPS, ACM và tên miền tùy chỉnh |
| Dữ liệu và AI | MongoDB Atlas, xử lý document/OCR và Groq |
| Vận hành | GitHub OIDC, CI/CD, CloudWatch Logs/Alarms và SNS |

Workshop tập trung vào kiến trúc production đang triển khai tại `ap-southeast-1`. Việc phát triển chi tiết từng màn hình không được lặp lại; phần này trình bày cách đưa ứng dụng hoàn chỉnh lên AWS và xác minh kết quả.

#### Công nghệ sử dụng

| Thành phần | Công nghệ | Trách nhiệm |
| --- | --- | --- |
| Frontend | React 18, TypeScript, Vite, Tailwind CSS | SPA theo vai trò, khóa học, quiz, upload và AI Assistant |
| Backend | Node.js 24, Express 5, Docker | REST API, authentication, nghiệp vụ, presigned URL và AI orchestration |
| Database | MongoDB Atlas, Mongoose | User, Course, Lesson, Enrollment, Quiz, Attempt, Progress và AI Message |
| Media | Amazon S3 | Video, document, thumbnail và avatar |
| AI | Groq API | Chat theo bài học, tóm tắt document và sinh quiz |
| Document | pdf-parse, Mammoth, Tesseract.js | Trích xuất PDF/DOCX và OCR PDF scan |

#### Chức năng chính

* Học viên đăng ký khóa học, học theo bài, xem tài liệu, theo dõi tiến độ và làm quiz.
* Giảng viên quản lý khóa học, bài học, học viên, quiz và xem chi tiết kết quả.
* Quản trị viên quản lý tài khoản, giám sát hệ thống và xem dữ liệu theo giảng viên.
* Media dung lượng lớn được upload trực tiếp bằng presigned URL và multipart upload.
* AI sử dụng nội dung document làm context; summary được lưu lại để tránh gọi model lặp lại.

#### Mục tiêu kỹ thuật

1. Triển khai Frontend private trên S3 và phân phối bằng CloudFront HTTPS.
2. Chạy Backend trên ít nhất hai EC2 private thuộc hai AZ.
3. Chỉ cho phép Backend nhận traffic từ ALB Security Group.
4. Duy trì Backend bằng ASG và health check `/health/ready`.
5. Phát hành image bất biến theo Git commit SHA.
6. Không lưu AWS access key dài hạn trong repository hoặc server.
7. Có log tập trung, cảnh báo và quy trình rollback.

#### Tổ chức repository

```text
LearnSphere/
├── LearnSphere_BE/          # Express API, models, services, Dockerfile
├── LearnSphere_FE/          # React/Vite SPA
├── .github/workflows/       # CI/CD production
├── docs/                    # Tài liệu triển khai
└── README.md
```

#### Sản phẩm production

![Trang chủ LearnSphere production](/images/learnsphere-production-homepage.png)

*Hình 5.2. Giao diện LearnSphere hoạt động trên tên miền production.*

Sau workshop, người thực hiện có thể:

* Triển khai lại kiến trúc Multi-AZ của LearnSphere.
* Phát hành phiên bản mới bằng `git push`.
* Xác minh ASG có hai instance và Target Group có hai target khỏe mạnh.
* Kiểm tra Frontend, API, S3 Media, MongoDB và Groq theo luồng end-to-end.
* Truy cập ứng dụng tại [https://www.learnspherev2.id.vn](https://www.learnspherev2.id.vn).
