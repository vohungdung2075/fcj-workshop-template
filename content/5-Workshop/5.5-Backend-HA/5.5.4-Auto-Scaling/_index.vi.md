---
title: "Auto Scaling Group Multi-AZ"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
---

#### Mục tiêu

Auto Scaling Group chịu trách nhiệm duy trì số lượng Backend instance, phân bố chúng qua hai Availability Zone và thay instance không healthy. Đây là thành phần loại bỏ single point of failure tại tầng compute.

#### 1. Tạo Auto Scaling Group

Tạo `LearnSphere-Backend-ASG`:

| Cấu hình | Giá trị |
| --- | --- |
| Launch Template | `LearnSphere-Backend-LT`, Default version |
| Private subnet 1a | `subnet-04724edeb47832763` |
| Private subnet 1b | `subnet-0ccb5a5e29560fa75` |
| Target Group | `LearnSphere-Backend-TG` |
| Minimum capacity | 2 |
| Desired capacity | 2 |
| Maximum capacity | 4 |

Với desired capacity 2 và hai private subnet, ASG duy trì một Backend instance trong mỗi AZ khi capacity và hạ tầng cho phép.

#### 2. Cấu hình health check

| Thuộc tính | Giá trị |
| --- | --- |
| Health check types | EC2 và ELB |
| Health check grace period | 360 giây |
| Default instance warmup | 300 giây |
| Default cooldown | 300 giây |

EC2 health check phát hiện lỗi hạ tầng máy ảo. ELB health check phát hiện container không chạy, port 5000 không lắng nghe hoặc readiness không đạt. Kết hợp hai loại giúp ASG không giữ một instance “Running” nhưng ứng dụng không phục vụ được.

#### 3. Instance maintenance policy

Chính sách:

| Thuộc tính | Giá trị |
| --- | --- |
| Replacement behavior | Launch before terminating |
| Minimum healthy percentage | 100% |
| Maximum healthy percentage | 200% |

Khi thay instance, ASG có thể tạm thời chạy thêm capacity. Instance cũ chỉ bị loại sau khi instance mới khởi động và Target Group xác nhận healthy.

#### 4. Service-linked role

ASG sử dụng:

```text
AWSServiceRoleForAutoScaling
```

Lần đầu tạo có thể xuất hiện thông báo service-linked role chưa sẵn sàng. Đây là độ trễ khi AWS tạo và propagate role; chờ một khoảng ngắn rồi thực hiện lại.

#### 5. Capacity và scaling

`minimum=2` bảo đảm ASG không scale xuống dưới hai Backend trong trạng thái bình thường. `desired=2` là capacity vận hành hiện tại. `maximum=4` chỉ đặt trần.

Nếu chưa có Dynamic scaling policy, ASG không tự tăng từ 2 lên 3 hoặc 4 theo CPU. Giai đoạn tiếp theo có thể tạo Target Tracking theo:

* ALB RequestCountPerTarget.
* Average CPUUtilization.
* Custom response-time hoặc queue metric.

#### 6. Xác minh

ASG Capacity Overview phải cho thấy:

```text
Desired capacity: 2
Minimum capacity: 2
Maximum capacity: 4
Healthy: 2/2
```

Tab Instance management phải có hai instance `InService`, `Healthy`, sử dụng Launch Template version 2 và nằm ở hai Availability Zone khác nhau.

#### Bằng chứng triển khai

![Capacity của Auto Scaling Group Backend LearnSphere](/images/learnsphere-asg-capacity.png)

*Hình 5.23. `LearnSphere-Backend-ASG` đang đạt desired capacity với hai instance và giới hạn scale từ 2–4. Nhóm sử dụng `LearnSphere-Backend-LT`, duy trì tối thiểu hai Backend instance trong điều kiện vận hành bình thường.*

![Hai Backend instance healthy phân bố qua hai Availability Zone](/images/learnsphere-asg-instances-multiaz.png)

*Hình 5.24. Instance management hiển thị hai instance `t3.small` ở trạng thái `InService`, `Healthy`. Hai instance được phân bố giữa `apse1-az2` (`ap-southeast-1a`) và `apse1-az1` (`ap-southeast-1b`), xác nhận cấu hình Multi-AZ.*
