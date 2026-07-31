---
title: "Kiểm tra Frontend production"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.6.4. </b> "
---

#### 1. Kiểm tra DNS và HTTPS

```powershell
nslookup www.learnspherev2.id.vn
curl.exe -I https://www.learnspherev2.id.vn
curl.exe -i https://www.learnspherev2.id.vn/api/courses
curl.exe -i https://origin.learnspherev2.id.vn/health/ready
```

Kết quả mong đợi:

* Domain phân giải đến CloudFront.
* Website trả HTTP 200 qua HTTPS.
* `/api/courses` được CloudFront chuyển đến Express và trả JSON.
* Origin `/health/ready` trả `{"status":"ready","database":"connected"}`.
* Trình duyệt không báo lỗi certificate hoặc mixed content.

#### 2. Kiểm tra SPA

Mở trực tiếp và refresh các đường dẫn như:

```text
/profile
/courses
/system-monitoring
/lesson-management
```

Mỗi route phải tải lại ứng dụng thay vì trả 403/404 từ S3. Một URL API không tồn tại phải trả JSON 404 từ Express, không được trả `index.html`.

#### 3. Kiểm tra cache và bản phát hành

Sau deploy:

1. Xác nhận CloudFront invalidation hoàn tất.
2. Mở cửa sổ ẩn danh để tránh service/cache cũ.
3. Kiểm tra tên file asset mới trong Developer Tools.
4. Đăng nhập và gọi một API có cookie.
5. Kiểm tra upload presigned URL vẫn tuân thủ S3 CORS.

#### Bằng chứng triển khai

![Giao diện LearnSphere hoạt động trên tên miền production](/images/learnsphere-production-homepage.png)

*Hình 5.36. LearnSphere được phân phối qua HTTPS trên môi trường production. Ảnh giao diện xác minh rằng CloudFront đã tải Frontend từ S3 và người dùng có thể truy cập ứng dụng bằng trình duyệt.*


![Các CloudFront invalidation đã hoàn tất](/images/learnsphere-cloudfront-invalidations.png)

*Hình 5.37. Lịch sử CloudFront Invalidations cho thấy các yêu cầu làm mới edge cache đều đã hoàn tất với trạng thái `Completed`.*

![Request API production đi qua cùng domain và trả về 200 OK](/images/learnsphere-production-api-network.png)

*Hình 5.38. Request `GET /api/users/me/courses` được gửi qua `https://www.learnspherev2.id.vn` và trả về `200 OK`, xác nhận CloudFront định tuyến API đến Backend thành công.*
