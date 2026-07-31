---
title: "Security Group và xác minh hạ tầng"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

#### Mục tiêu

Security Group của LearnSphere được thiết kế theo chuỗi tin cậy:

```text
Internet / CloudFront
→ HTTPS:443
→ LearnSphere-ALB-SG
→ TCP:5000
→ LearnSphere-Backend-SG
→ Backend EC2
```

Người dùng không thể kết nối trực tiếp tới port 5000 của Backend. EC2 cũng không có public IP và không mở SSH.

#### 1. ALB Security Group

Tạo `LearnSphere-ALB-SG` (`sg-0f2fe594908268fc3`) trong `LearnSphere-Prod-vpc`.

Inbound hiện tại:

| Type | Protocol | Port | Source | Mục đích |
| --- | --- | --- | --- | --- |
| HTTPS | TCP | 443 | Internet | Nhận HTTPS từ CloudFront và health validation |

ALB listener dùng ACM certificate cho `origin.learnspherev2.id.vn`. Không mở HTTP port 80 nếu không cần redirect HTTP sang HTTPS.

Trong tương lai, sau khi pipeline health check đi qua CloudFront, inbound của ALB có thể giới hạn bằng AWS-managed prefix list cho CloudFront origin-facing servers.

#### 2. Backend Security Group

Tạo `LearnSphere-Backend-SG` (`sg-0cae310ddce032bbf`) trong cùng VPC.

Inbound:

| Type | Protocol | Port | Source |
| --- | --- | --- | --- |
| Custom TCP | TCP | 5000 | `sg-0f2fe594908268fc3` |

Source là Security Group ID của ALB, không phải CIDR `0.0.0.0/0` và không phải public IP của ALB. Khi ALB node thay đổi IP, rule vẫn hoạt động.

Không thêm inbound SSH 22. Systems Manager Agent dùng kết nối outbound HTTPS để quản trị instance.

#### 3. Outbound và dependency bên ngoài

Backend cần outbound để:

* Đọc Parameter Store và giao tiếp Systems Manager.
* Đăng nhập và pull image từ ECR.
* Gửi log tới CloudWatch Logs.
* Truy cập S3 Media qua S3 Gateway Endpoint.
* Kết nối MongoDB Atlas bằng TLS.
* Gọi Groq API và dịch vụ email bằng HTTPS.

Private route table quyết định traffic đi S3 Endpoint hay NAT Gateway. Security Group là stateful nên response traffic của kết nối hợp lệ được tự động cho phép.

#### 4. Bảo vệ database và object storage

MongoDB Atlas chỉ allowlist:

```text
54.179.11.158/32
52.221.42.74/32
```

Không dùng `0.0.0.0/0` cho production. Database credential được lưu trong `/learnsphere/prod/backend-env`, không đặt trong Security Group, User Data hoặc GitHub.

Hai S3 bucket tiếp tục bật Block Public Access. S3 Frontend chỉ cho CloudFront OAC đọc; S3 Media chỉ được truy cập bằng EC2 role hoặc presigned URL có thời hạn.

#### 5. Kiểm tra đường truy cập

Thực hiện các kiểm tra sau:

| Kiểm tra | Kết quả mong đợi |
| --- | --- |
| `https://www.learnspherev2.id.vn` | HTTP 200 và tải Frontend |
| `https://origin.learnspherev2.id.vn/health/ready` | HTTP 200 khi Target Group healthy |
| Public IP của Backend | Không tồn tại |
| Kết nối trực tiếp `<private-ip>:5000` từ Internet | Không thể thực hiện |
| Truy cập Media object không presigned URL | AccessDenied |
| SSM Managed nodes | Hai EC2 hiển thị Online |
| MongoDB Atlas allowlist | Chỉ có hai NAT Elastic IP production |

Lệnh xác minh health endpoint:

```powershell
curl.exe -i https://origin.learnspherev2.id.vn/health/ready
```

Response mong đợi:

```json
{"status":"ready","database":"connected"}
```

#### 6. Checklist trước khi sang mục 5.5

* ALB và Backend Security Group thuộc đúng VPC production.
* Backend inbound port 5000 chỉ có source là ALB Security Group.
* Không có rule SSH từ Internet.
* Hai private subnet không auto-assign public IPv4.
* NAT Elastic IP trùng với Atlas allowlist.
* EC2 instance role có đủ quyền nhưng không dùng AdministratorAccess.
* S3 bucket không public.
* SecureString không xuất hiện trong ảnh hoặc log.

#### Kết quả cấu hình bảo mật mạng

![Inbound rule của ALB Security Group](/images/learnsphere-alb-security-group.png)

*Hình 5.16. `LearnSphere-ALB-SG` thuộc production VPC và chỉ có một inbound rule HTTPS port 443. ALB không mở trực tiếp port ứng dụng 5000.*

![Inbound rule của Backend Security Group](/images/learnsphere-backend-security-group.png)

*Hình 5.17. `LearnSphere-Backend-SG` chỉ cho phép TCP port 5000 từ Security Group `LearnSphere-ALB-SG` (`sg-0f2fe594908268fc3`), thay vì mở port Backend cho toàn bộ Internet.*

![Thông tin mạng của Backend EC2 private](/images/learnsphere-private-ec2-network.png)

*Hình 5.18. Backend EC2 nằm trong private subnet tại `ap-southeast-1b`, có private IPv4 `10.20.153.245` nhưng không có public IPv4, public DNS hoặc IPv6 public.*

Ba lớp kiểm soát bổ sung cho nhau: ALB là điểm vào HTTPS duy nhất, Backend chỉ tin cậy traffic từ ALB Security Group, còn EC2 không có địa chỉ public để Internet kết nối trực tiếp. Quản trị instance được thực hiện qua Systems Manager thay vì SSH.
