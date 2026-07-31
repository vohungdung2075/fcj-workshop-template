---
title: "Triển khai Backend High Availability"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Mục tiêu

Mục 5.4 đã hoàn thành IAM, lưu trữ và mạng Multi-AZ. Mục 5.5 sử dụng nền hạ tầng đó để triển khai Backend Node.js/Express theo mô hình High Availability gồm Application Load Balancer và Auto Scaling Group trên hai Availability Zone.

Kết quả cần đạt:

* Backend chạy bằng Docker image bất biến từ Amazon ECR.
* Secret production không nằm trong Docker image hoặc Launch Template.
* Hai EC2 Backend chạy trong hai private subnet khác nhau.
* ALB chỉ phân phối request tới target vượt qua readiness check.
* ASG duy trì tối thiểu hai instance và tự thay instance lỗi.
* Instance mới có thể tự bootstrap mà không cần SSH hoặc cấu hình thủ công.
* Quá trình thay phiên bản sử dụng launch-before-terminate để giữ capacity.

#### Các thành phần production

| Thành phần | Tài nguyên |
| --- | --- |
| Runtime configuration | `/learnsphere/prod/backend-env` |
| Released image tag | `/learnsphere/prod/backend-image-tag` |
| Application Load Balancer | `LearnSphere-Prod-ALB` |
| Target Group | `LearnSphere-Backend-TG` |
| Launch Template | `LearnSphere-Backend-LT`, default version `2` |
| Auto Scaling Group | `LearnSphere-Backend-ASG` |
| Backend instances | Hai EC2 `t3.small` trong hai private subnet |
| Health endpoint | `GET /health/ready` |
| Backend origin | `https://origin.learnspherev2.id.vn` |

#### Các bước triển khai

1. [Chuẩn bị cấu hình runtime trong Parameter Store](5.5.1-runtime-configuration/)
2. [Tạo Target Group và Application Load Balancer](5.5.2-alb-target-group/)
3. [Tạo Launch Template và User Data bootstrap](5.5.3-launch-template/)
4. [Tạo Auto Scaling Group Multi-AZ](5.5.4-auto-scaling/)
5. [Xác minh High Availability và self-healing](5.5.5-ha-validation/)

#### Luồng khởi tạo một Backend instance

```text
Auto Scaling Group
→ chọn private subnet
→ khởi tạo EC2 từ Launch Template
→ EC2 đọc environment và image tag từ Parameter Store
→ EC2 đăng nhập ECR bằng instance role
→ pull Docker image theo commit SHA
→ chạy container trên port 5000
→ gửi log đến CloudWatch
→ trả HTTP 200 tại /health/ready
→ Target Group chuyển target thành Healthy
→ ALB bắt đầu gửi request
```

#### Tiêu chí hoàn thành

Backend HA được xem là hoàn tất khi hai instance đều `InService` và `Healthy`, Target Group có hai target healthy thuộc hai Availability Zone, ALB trả HTTP 200 cho readiness endpoint và một instance có thể bị thay thế mà không làm mất toàn bộ khả năng phục vụ.

Giá trị ASG `maximum=4` là giới hạn capacity, không tự động tạo scale-out. Kiến trúc hiện cung cấp HA và self-healing ở desired capacity 2; tự động mở rộng theo tải cần bổ sung Target Tracking hoặc Step Scaling policy trong giai đoạn tiếp theo.
