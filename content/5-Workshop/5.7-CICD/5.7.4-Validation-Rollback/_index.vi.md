---
title: "Xác minh release và rollback"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.7.4. </b> "
---

#### 1. Xác minh Backend sau refresh

Workflow kiểm tra:

```text
ASG desired capacity == số instance InService
ASG healthy capacity == desired capacity
Instance Refresh == Successful
GET https://origin.learnspherev2.id.vn/health/ready == HTTP 200
```

Readiness endpoint còn xác nhận MongoDB đang connected, do đó một container chỉ “running” nhưng chưa dùng được database không được xem là release thành công.

#### 2. Cơ chế rollback

Trước khi ghi tag mới, pipeline lưu `OLD_IMAGE_TAG`. Nếu refresh thất bại hoặc public health check không đạt:

1. Ghi lại tag cũ vào Parameter Store.
2. Khởi chạy một Instance Refresh mới.
3. Các EC2 mới bootstrap bằng image cũ đã biết là hoạt động.
4. Workflow kết thúc thất bại để người vận hành điều tra.

Rollback dùng cùng đường triển khai với release thường, tránh thao tác SSH sửa riêng từng instance.

#### 3. Kiểm tra toàn bộ workflow

Sau một run thành công cần đối chiếu:

| Kiểm tra | Kết quả |
| --- | --- |
| Backend tests | Pass |
| ECR image tag | Trùng commit SHA |
| Instance Refresh | Successful |
| Target Group | 2 healthy, 0 unhealthy |
| Origin readiness | HTTP 200 |
| Frontend build | Success |
| S3 publish | Success |
| CloudFront invalidation | Completed |

#### Bằng chứng triển khai

![Hai job triển khai LearnSphere hoàn tất trên GitHub Actions](/images/learnsphere-github-actions-overview.png)

*Hình 5.45. Workflow production hoàn tất tuần tự Backend và Frontend. Trạng thái xanh của cả hai job là bằng chứng tổng hợp, nhưng vẫn phải kết hợp với health check và trạng thái AWS thay vì chỉ dựa vào giao diện GitHub.*


![Lịch sử Instance Refresh thành công của Backend ASG](/images/learnsphere-asg-instance-refresh-history.png)

*Hình 5.46. Lịch sử Auto Scaling cho thấy ba lần Instance Refresh đều đạt trạng thái `Successful` và hoàn tất 100%, xác nhận cơ chế rollout thay thế instance hoạt động ổn định.*

Không cần tạo lỗi production để chụp rollback. Có thể dùng sơ đồ hoặc code snippet nhánh rollback và mô tả đây là control đã được cài đặt.
