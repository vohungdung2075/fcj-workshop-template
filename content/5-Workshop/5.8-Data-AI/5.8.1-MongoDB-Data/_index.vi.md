---
title: "MongoDB Atlas và dữ liệu dùng chung"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.8.1. </b> "
---

#### 1. Vai trò của MongoDB Atlas

Backend dùng Mongoose và các document schema cho:

* Tài khoản, role và hồ sơ.
* Khóa học, bài học và enrollment.
* Quiz, question, attempt và kết quả.
* Notification, upload session và AI rate-limit bucket.
* AI message, indexed text, summary và token usage.

Database nằm ngoài vòng đời EC2. Khi ASG thay instance, dữ liệu nghiệp vụ không bị mất và instance mới kết nối vào cùng cluster.

#### 2. Kết nối từ private subnet

```text
EC2 private subnet 1a → NAT Gateway 1a ┐
                                      ├→ MongoDB Atlas TLS
EC2 private subnet 1b → NAT Gateway 1b ┘
```

Atlas Network Access allowlist hai NAT Elastic IP `/32`, không allowlist private IP động của từng EC2. Connection string nằm trong `/learnsphere/prod/backend-env` dưới dạng SecureString.

Readiness endpoint chỉ trả `ready` khi database đang connected:

```json
{
  "status": "ready",
  "database": "connected"
}
```

#### 3. Tính nhất quán trong HA

Hai instance có thể phục vụ request đồng thời vì session xác thực nằm trong cookie/JWT và state bền vững nằm trong MongoDB/S3. Không lưu enrollment, quiz attempt hoặc AI history trong RAM của một instance cụ thể.

`MONGODB_REQUIRE_TRANSACTIONS` cho phép production yêu cầu transaction ở các nghiệp vụ nhiều bước khi cluster hỗ trợ. Unique index và atomic update tiếp tục bảo vệ race condition như AI rate limiting và summary generation.

#### 4. Lựa chọn hiện tại và hướng phát triển

MongoDB Atlas phù hợp với code hiện tại vì Mongoose schema và các nghiệp vụ đã hoàn thiện. Chuyển ngay sang DynamoDB sẽ cần thiết kế lại partition key, access pattern, transaction và migration.

Trong giai đoạn mở rộng, nhóm sẽ đánh giá DynamoDB theo từng workload có access pattern rõ ràng, lưu lượng lớn hoặc cần serverless scaling; không thực hiện thay thế cơ học toàn bộ MongoDB. Các cải tiến khác gồm Atlas Private Endpoint/peering, backup policy, multi-region database và kiểm thử khôi phục.

#### Bằng chứng triển khai

![MongoDB Atlas IP Access List chỉ cho phép hai địa chỉ public IP của NAT Gateway](/images/learnsphere-mongodb-nat-ip-access-list.png)

**Hình 5.47:** MongoDB Atlas Network Access chỉ cho phép hai public IP `/32` của NAT Gateway tại `ap-southeast-1a` và `ap-southeast-1b`. Cả hai bản ghi đều ở trạng thái Active, trong khi connection string và thông tin người dùng không được hiển thị.

![Các collection MongoDB Atlas được Backend LearnSphere sử dụng](/images/learnsphere-mongodb-collections.png)

**Hình 5.48:** Atlas Collections thể hiện mô hình dữ liệu bền vững dùng chung cho hai Backend instance, gồm khóa học, bài học, đăng ký, tiến độ, lượt làm quiz, thông báo, tin nhắn AI và bucket giới hạn request AI. Ảnh không mở document chứa dữ liệu cá nhân.
