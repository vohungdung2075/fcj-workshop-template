---
title: "Workshop"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai LearnSphere trên AWS

#### Tổng quan

Workshop trình bày toàn bộ quá trình đưa **LearnSphere** từ mã nguồn local lên môi trường production tại AWS Region Singapore (`ap-southeast-1`). Frontend React/Vite được phân phối bằng Amazon S3 và CloudFront; Backend Node.js/Express được đóng gói Docker, triển khai trên Auto Scaling Group gồm hai EC2 private thuộc hai Availability Zone và phục vụ qua Application Load Balancer.

Quy trình CI/CD hiện tại dùng GitHub OIDC, ECR, Systems Manager Parameter Store và Auto Scaling Instance Refresh có kiểm tra sức khỏe và khôi phục image tag khi phát hành thất bại.

#### Kết quả chính

* Website production: [https://www.learnspherev2.id.vn](https://www.learnspherev2.id.vn).
* Backend hoạt động trong hai private subnet thuộc hai Availability Zone.
* ALB phân phối request API đến hai target khỏe mạnh.
* ASG duy trì `min=2`, `desired=2`, `max=4`.
* Frontend và media được lưu trong hai S3 bucket private riêng biệt.
* MongoDB Atlas và Groq được truy cập qua NAT Gateway theo từng AZ.
* GitHub Actions triển khai bằng credential tạm thời từ OIDC, không dùng AWS access key dài hạn.
* CloudWatch tập trung log; SNS gửi cảnh báo tới quản trị viên.

#### Nội dung

1. [Tổng quan dự án](5.1-overview/)
2. [Chuẩn bị triển khai](5.2-preparation/)
3. [Kiến trúc và luồng hệ thống](5.3-architecture/)
4. [Xây dựng hạ tầng AWS cốt lõi](5.4-infrastructure/)
5. [Triển khai Backend High Availability](5.5-backend-ha/)
6. [Triển khai Frontend, CloudFront và tên miền](5.6-frontend-domain/)
7. [Tự động hóa CI/CD](5.7-cicd/)
8. [Dữ liệu, media và AI](5.8-data-ai/)
9. [Giám sát và cảnh báo](5.9-monitoring/)
10. [Kiểm thử và kết quả](5.10-testing/)
11. [Phân tích chi phí](5.11-cost/)
12. [Dọn dẹp tài nguyên](5.12-cleanup/)

#### Kết luận

Workshop hoàn thiện quy trình triển khai LearnSphere từ mã nguồn lên môi trường production theo hướng bảo mật, tự động hóa và sẵn sàng cao. Hệ thống kết hợp CloudFront, S3, ALB và Auto Scaling Group trên hai Availability Zone; Backend được phát hành bằng GitHub Actions, GitHub OIDC và ECR mà không cần quản lý SSH hoặc Access Key dài hạn. Kết quả đạt được là một nền tảng học tập trực tuyến có thể vận hành ổn định, theo dõi tập trung và tiếp tục mở rộng khi số lượng người dùng tăng.
