---
title: "Xây dựng hạ tầng AWS cốt lõi"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Mục tiêu

Phần này trình bày cách tạo lớp hạ tầng nền tảng trước khi triển khai Backend High Availability ở mục 5.5. Các tài nguyên được thiết kế theo nguyên tắc tách quyền, tách lớp mạng và không công khai trực tiếp dữ liệu hoặc máy chủ ứng dụng.

Sau khi hoàn thành mục 5.4, môi trường production có:

* GitHub Actions nhận AWS credential tạm thời qua OIDC, không lưu Access Key dài hạn.
* EC2 sử dụng instance profile riêng để đọc cấu hình, tải image và truy cập học liệu.
* Hai S3 bucket private tách biệt Frontend và Media.
* Một ECR repository quản lý Docker image Backend theo Git commit SHA.
* Một VPC trải trên hai Availability Zone, gồm hai public subnet và hai private subnet.
* Hai NAT Gateway cung cấp egress độc lập theo từng Availability Zone.
* S3 Gateway Endpoint giúp traffic S3 từ private subnet không đi qua NAT Gateway.
* Security Group chỉ cho phép ALB truy cập Backend qua port 5000.

#### Phạm vi tài nguyên

| Nhóm | Tài nguyên production |
| --- | --- |
| AWS Account | `440893644584` |
| Region | Singapore (`ap-southeast-1`) |
| VPC | `LearnSphere-Prod-vpc` — `10.20.0.0/16` |
| Public subnet | Hai subnet tại `ap-southeast-1a` và `ap-southeast-1b` |
| Private subnet | Hai subnet tại `ap-southeast-1a` và `ap-southeast-1b` |
| Container registry | ECR repository `learnsphere-be-2` |
| Static Frontend | S3 bucket `learnsphere-fe-2` |
| Media | S3 bucket `learnsphere-media-2` |
| Deployment identity | `LearnSphereGitHubDeployRole2` |
| Runtime identity | `LearnSphereEc2Role2` |
| Network security | `LearnSphere-ALB-SG`, `LearnSphere-Backend-SG` |

#### Các bước thực hiện

1. [Thiết lập IAM, GitHub OIDC và quyền runtime](5.4.1-iam/)
2. [Tạo S3 và Amazon ECR](5.4.2-storage-ecr/)
3. [Xây dựng mạng VPC Multi-AZ](5.4.3-network/)
4. [Cấu hình Security Group và xác minh nền hạ tầng](5.4.4-security/)

Thứ tự trên được lựa chọn để tài nguyên tạo sau có thể tham chiếu đúng role, bucket, repository, subnet và Security Group đã tồn tại. Backend HA, Launch Template, ALB, Target Group và Auto Scaling Group được triển khai ở mục 5.5.

#### Tiêu chí hoàn thành

Hạ tầng cốt lõi được xem là sẵn sàng khi:

* GitHub OIDC trust policy chỉ cho phép đúng repository và nhánh `main`.
* Hai S3 bucket vẫn bật Block Public Access.
* ECR đã có Docker image được gắn tag bằng commit SHA.
* Hai private subnet có route mặc định tới NAT Gateway cùng AZ.
* Hai private route table cùng sử dụng S3 Gateway Endpoint.
* Backend Security Group không cho phép `0.0.0.0/0` truy cập port 5000.
* Hai NAT Elastic IP đã được allowlist trong MongoDB Atlas.

#### Kết quả

Hạ tầng sau mục này tạo một ranh giới rõ ràng giữa edge, public ingress, private compute và các dịch vụ dữ liệu. Các EC2 Backend có thể được Auto Scaling Group tạo hoặc thay thế mà không cần public IP, SSH key hay cấu hình thủ công trên từng máy.
