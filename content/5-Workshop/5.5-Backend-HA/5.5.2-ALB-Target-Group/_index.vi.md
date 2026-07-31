---
title: "Target Group và Application Load Balancer"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---

#### Mục tiêu

Application Load Balancer tạo một HTTPS entry point ổn định cho nhiều Backend instance có vòng đời thay đổi. Target Group kiểm tra readiness và chỉ chuyển request tới instance có ứng dụng cùng database sẵn sàng.

#### 1. Tạo Target Group

Tạo `LearnSphere-Backend-TG`:

| Thuộc tính | Giá trị |
| --- | --- |
| Target type | Instance |
| Protocol | HTTP |
| Port | 5000 |
| Protocol version | HTTP1 |
| VPC | `vpc-0d59e8bb67e90a0da` |

Health check:

| Thuộc tính | Giá trị |
| --- | --- |
| Protocol | HTTP |
| Port | Traffic port |
| Path | `/health/ready` |
| Success code | `200` |

Readiness endpoint chỉ trả HTTP 200 khi tiến trình Node.js chạy và MongoDB đã kết nối:

```json
{
  "status": "ready",
  "database": "connected"
}
```

Nếu database mất kết nối, endpoint không báo ready. ALB loại target khỏi rotation thay vì tiếp tục gửi request tới một Backend không thể xử lý nghiệp vụ.

#### 2. Tạo Application Load Balancer

Tạo `LearnSphere-Prod-ALB`:

| Thuộc tính | Giá trị |
| --- | --- |
| Scheme | Internet-facing |
| IP address type | IPv4 |
| VPC | `LearnSphere-Prod-vpc` |
| Subnet | Public 1a và Public 1b |
| Security Group | `LearnSphere-ALB-SG` |

ALB được triển khai trên hai public subnet để có node tại cả hai Availability Zone. Backend EC2 vẫn nằm trong private subnet.

#### 3. Cấu hình HTTPS listener

Listener production:

```text
Protocol: HTTPS
Port: 443
Certificate: origin.learnspherev2.id.vn
Default action: Forward to LearnSphere-Backend-TG
```

ACM certificate của ALB phải được tạo tại `ap-southeast-1`, cùng Region với ALB. DNS TenTen có CNAME:

```text
origin.learnspherev2.id.vn
→ LearnSphere-Prod-ALB-1917416022.ap-southeast-1.elb.amazonaws.com
```

ALB terminate TLS và chuyển tiếp HTTP port 5000 trong VPC. Security Group của Backend chỉ chấp nhận source là ALB Security Group.

#### 4. Gắn Target Group với ASG

Không đăng ký thủ công các instance dài hạn. `LearnSphere-Backend-ASG` được gắn trực tiếp với Target Group:

* Instance mới được tự động register.
* Instance terminate được tự động deregister.
* ALB chỉ sử dụng target healthy.
* ASG có thể dùng ELB health check để thay instance lỗi.

#### 5. Kiểm tra

Trước khi ASG có instance, ALB có thể trả HTTP 503 vì Target Group chưa có target healthy. Sau khi hai Backend bootstrap thành công:

```powershell
curl.exe -i https://origin.learnspherev2.id.vn/health/ready
```

Response phải là HTTP 200. ALB listener phải forward 100% traffic tới `LearnSphere-Backend-TG`.

#### Bằng chứng triển khai

![Network mapping của ALB LearnSphere trên hai public subnet](/images/learnsphere-alb-network-mapping.png)

*Hình 5.20. `LearnSphere-Prod-ALB` được ánh xạ vào hai public subnet tại `ap-southeast-1a` và `ap-southeast-1b` trong VPC `10.20.0.0/16`, tạo đường ingress dự phòng qua hai Availability Zone.*

![HTTPS listener và quy tắc chuyển tiếp Target Group của ALB LearnSphere](/images/learnsphere-alb-https-listener.png)

*Hình 5.21. ALB lắng nghe HTTPS port 443, sử dụng certificate cho `origin.learnspherev2.id.vn` và chuyển tiếp 100% traffic phù hợp tới `LearnSphere-Backend-TG`.*
