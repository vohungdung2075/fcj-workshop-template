---
title: "Giám sát và cảnh báo"
date: 2026-07-30
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

#### Mục tiêu giám sát

Hệ thống giám sát production cần trả lời được bốn câu hỏi vận hành:

1. Ứng dụng có thể truy cập qua CloudFront và ALB hay không?
2. Cả hai Backend target có sẵn sàng nhận request hay không?
3. Ứng dụng có phát sinh lỗi, phản hồi chậm hoặc lỗi nhà cung cấp AI hay không?
4. Hệ thống có tự phục hồi khi một EC2 instance mất khả năng phục vụ hay không?

Phạm vi giám sát vì vậy bao gồm CloudFront, ALB, Target Group, Auto Scaling Group, container trên EC2, trạng thái kết nối MongoDB và lời gọi tới nhà cung cấp AI. CloudWatch tập trung metric và application log; SNS chuyển cảnh báo đến email của quản trị viên.

| Nguồn tín hiệu | Bằng chứng chính | Mục đích vận hành |
| --- | --- | --- |
| ALB/Target Group | target health, response time, HTTP 5xx | Phát hiện Backend ngừng hoạt động hoặc suy giảm |
| Auto Scaling Group | desired, in-service, pending instances | Xác nhận capacity và khả năng tự phục hồi |
| EC2/Docker | stdout/stderr của ứng dụng | Điều tra lỗi request và runtime |
| `/health/ready` | trạng thái ứng dụng và database | Ngăn traffic đi vào target chưa sẵn sàng |
| Tích hợp AI | mã lỗi đã chuẩn hóa trong log | Phân biệt quota, timeout, credential và provider failure |

#### Tập trung log bằng CloudWatch

Docker dùng `awslogs` logging driver để đưa log của từng EC2 vào log group:

```text
/learnsphere/backend2
```

Mỗi instance/container có log stream riêng. Khi ASG thay thế instance, log cũ vẫn được giữ tập trung nên việc điều tra không phụ thuộc vào SSH hoặc ổ đĩa cục bộ còn tồn tại. Log group cần có retention period phù hợp để kiểm soát chi phí lưu trữ và chính sách lưu dữ liệu.

Ví dụ truy vấn lỗi ứng dụng và AI gần nhất:

```sql
fields @timestamp, @message
| filter @message like /ERROR|Exception|AI_THROTTLED/
| sort @timestamp desc
| limit 100
```

Log không được chứa JWT, cookie, MongoDB connection string, mật khẩu email, Groq key, toàn bộ prompt hoặc presigned URL riêng tư. Identifier cần được che khi không cần thiết cho việc điều tra.

#### Chuỗi health check và thay thế instance

Ứng dụng cung cấp liveness endpoint để kiểm tra process và readiness endpoint để quyết định khả năng nhận traffic production. `/health/ready` xác minh cả ứng dụng và database:

```json
{"status":"ready","database":"connected"}
```

ALB Target Group gọi readiness endpoint trên port `5000`. Chỉ target Healthy mới nhận traffic. ASG đồng thời sử dụng ELB health status; instance thất bại lặp lại sẽ bị loại bỏ và tạo lại từ Launch Template mặc định. Chuỗi tự phục hồi diễn ra như sau:

```text
Readiness thất bại
→ Target Group đánh dấu target Unhealthy
→ ALB ngừng chuyển request mới đến target
→ ASG thay thế instance lỗi
→ User data cài Docker và chạy image hiện tại
→ Target Healthy và quay lại phục vụ
```

#### CloudWatch Alarms và SNS

SNS topic `LearnSphere-Alerts` gửi email tới Admin. Trong kiến trúc ASG cuối cùng, alarm phải theo tài nguyên logic thay vì Instance ID cố định:

| Metric | Điều kiện ban đầu | Ý nghĩa |
| --- | --- | --- |
| ALB `UnHealthyHostCount` | `>= 1` trong hai chu kỳ | Có ít nhất một Backend target không sẵn sàng |
| ALB `HTTPCode_Target_5XX_Count` | vượt baseline lỗi được chấp nhận | Ứng dụng hoặc dependency gặp sự cố |
| ASG `GroupInServiceInstances` | `< 2` | Capacity HA thấp hơn mức tối thiểu |
| ALB `TargetResponseTime` | vượt ngưỡng latency liên tục | Backend suy giảm dù vẫn còn Healthy |
| CloudWatch Logs metric filter | có `ERROR`, `Exception` hoặc lỗi AI lặp lại | Sự cố tầng ứng dụng cần điều tra |

Alarm `CPUUtilization` hoặc `StatusCheckFailed` gắn với Instance ID cũ không đại diện cho instance được ASG tạo về sau. Các alarm này cần được loại bỏ hoặc thay bằng alarm theo ASG/ALB, giám sát động theo instance hoặc composite alarm.

#### Quy trình xử lý sự cố

Khi alarm chuyển sang `ALARM`, người vận hành thực hiện:

1. Xác định tài nguyên và khoảng thời gian lỗi từ alarm history.
2. Kiểm tra Target Group health và ASG activity trước khi thay đổi capacity thủ công.
3. Truy vấn `/learnsphere/backend2` trong cùng khoảng thời gian và đối chiếu log stream bị ảnh hưởng.
4. Kiểm tra `/health/ready`, sau đó phân biệt lỗi ứng dụng, database, mạng hoặc nhà cung cấp ngoài.
5. Cho phép ASG hoàn tất thay thế nếu sự cố chỉ nằm trên một instance.
6. Ghi lại nguyên nhân, thao tác phục hồi và công việc phòng ngừa sau khi hệ thống ổn định.

Các lệnh kiểm tra chỉ đọc:

```bash
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names LearnSphere-Backend-ASG

aws elbv2 describe-target-health \
  --target-group-arn <TARGET_GROUP_ARN>

aws logs tail /learnsphere/backend2 --since 15m
```

#### Bằng chứng triển khai

![Các CloudWatch log stream của Backend](/images/learnsphere-cloudwatch-log-streams.png)

*Hình 5.56. Các log stream trong CloudWatch Logs ghi nhận nhiều vòng triển khai và thay thế Backend instance. Tên stream bắt đầu bằng `asg-` cho phép đối chiếu log với từng EC2 private hostname mà không cần đăng nhập trực tiếp vào máy chủ.*

![Trạng thái CloudWatch Alarms](/images/learnsphere-cloudwatch-alarms.png)

*Hình 5.57a. Danh sách CloudWatch Alarms tại thời điểm kiểm tra, bao gồm alarm target tracking của Auto Scaling Group và các alarm EC2 cũ. Alarm Low của target tracking có thể chuyển sang `In alarm` khi tải thấp để kích hoạt scale-in; đây không đồng nghĩa ứng dụng gặp sự cố. Hai alarm gắn với EC2 cố định được giữ làm bằng chứng giai đoạn triển khai cũ và cần được thay bằng alarm theo ALB/ASG.*

![SNS subscription đã được xác nhận](/images/learnsphere-sns-confirmed-subscription.png)

*Hình 5.57b. SNS topic `LearnSphere-Alerts2` có email subscription ở trạng thái `Confirmed`, sẵn sàng nhận thông báo do CloudWatch Alarm gửi tới.*
