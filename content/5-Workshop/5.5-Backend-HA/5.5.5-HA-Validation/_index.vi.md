---
title: "Xác minh High Availability và self-healing"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5.5.5. </b> "
---

#### Mục tiêu

Việc có hai EC2 trên sơ đồ chưa đủ chứng minh HA. Cần kiểm tra đồng thời Target Group, ASG, readiness endpoint và lịch sử thay thế instance.

#### 1. Kiểm tra Target Group

Trong `LearnSphere-Backend-TG`, hai target phải:

* Có trạng thái `Healthy`.
* Thuộc hai Availability Zone khác nhau.
* Lắng nghe port 5000.
* Không có target `Unused`, `Initial`, `Draining` hoặc `Unhealthy` sau khi hệ thống ổn định.

Target Group dùng HTTP health check đến `/health/ready`; đây là kiểm tra ứng dụng, không chỉ kiểm tra EC2 đang bật.

#### 2. Kiểm tra public readiness

```powershell
curl.exe -i https://origin.learnspherev2.id.vn/health/ready
```

Kết quả:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

```json
{"status":"ready","database":"connected"}
```

Điều này xác nhận chuỗi DNS → HTTPS listener → ALB → Target Group → Backend → MongoDB hoạt động.

#### 3. Kiểm tra từng instance

Qua Systems Manager Run Command:

```bash
sudo docker ps --filter name=learnsphere-be
sudo docker inspect \
  --format 'status={{.State.Status}} restarts={{.RestartCount}} image={{.Config.Image}}' \
  learnsphere-be
curl -fsS http://127.0.0.1:5000/health/ready
```

Thực hiện trên cả hai instance. Hai container phải chạy cùng image tag production và readiness cùng trả `database=connected`.

#### 4. Xác minh self-healing

Có thể dùng lịch sử Instance Refresh hoặc một bài kiểm tra có kiểm soát:

1. Ghi nhận hai target đang healthy.
2. Chọn một instance thuộc ASG và terminate qua Auto Scaling/EC2.
3. Quan sát ALB ngừng gửi request tới target cũ.
4. ASG tạo instance thay thế từ Launch Template.
5. Instance mới bootstrap từ Parameter Store và ECR.
6. Target mới chuyển từ `Initial` sang `Healthy`.
7. Trong suốt quá trình, endpoint vẫn được phục vụ bởi target còn lại.

Chỉ thực hiện thử nghiệm terminate khi hai target ban đầu đều healthy và có đủ thời gian theo dõi. Không giảm desired capacity trong bài kiểm tra self-healing.

#### 5. Xác minh launch-before-terminate

Pipeline dùng Instance Refresh:

```text
MinHealthyPercentage: 100
MaxHealthyPercentage: 200
InstanceWarmup: 300
```

Workflow:

1. Kiểm tra candidate image tồn tại trong ECR.
2. Xác minh ASG hiện healthy và không có refresh khác đang chạy.
3. Lưu image tag cũ.
4. Cập nhật candidate tag vào Parameter Store.
5. Khởi chạy Instance Refresh.
6. Chờ refresh thành công và đủ healthy InService instance.
7. Gọi readiness endpoint qua ALB.
8. Nếu thất bại, khôi phục tag cũ và chạy rollback refresh.

#### 6. Phạm vi HA

Kiến trúc đạt HA trong một AWS Region và hai Availability Zone cho tầng Backend. Nó chống được lỗi một container hoặc một EC2 instance và duy trì đường phục vụ khi một target bị loại.

Đây chưa phải Disaster Recovery đa Region. MongoDB Atlas và Groq vẫn là external dependencies; độ sẵn sàng tổng thể phụ thuộc cấu hình cluster Atlas và dịch vụ Groq.

#### Bằng chứng kiểm thử

![Target Group Backend LearnSphere có hai target healthy](/images/learnsphere-target-group-healthy.png)

*Hình 5.25. `LearnSphere-Backend-TG` báo cáo tổng cộng hai target, cả hai đều healthy và không có target unhealthy trên HTTP port 5000. Kết hợp với cấu hình Multi-AZ ở Hình 5.24, kết quả xác nhận hai Backend instance đều đủ điều kiện nhận traffic từ ALB.*

![Readiness endpoint của LearnSphere trả HTTP 200](/images/learnsphere-readiness-200.png)

*Hình 5.26. Lời gọi `https://origin.learnspherev2.id.vn/health/ready` qua public ALB trả HTTP 200 cùng JSON `{"status":"ready","database":"connected"}`, xác nhận toàn bộ đường đi TLS, cân bằng tải, Backend và database hoạt động bình thường.*

![Hoạt động thay thế instance thành công của Auto Scaling Group LearnSphere](/images/learnsphere-asg-self-healing.png)

*Hình 5.27. ASG Activity history ghi nhận hoạt động terminate instance thành công trong quá trình Instance Refresh, sau đó nhóm trở lại desired capacity với 2/2 instance healthy. Đây là bằng chứng vận hành cho khả năng thay thế có kiểm soát và self-healing.*
