---
title: "Frontend, CloudFront và tên miền"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Mục tiêu

Mục này triển khai lớp phân phối Frontend của LearnSphere sau khi hạ tầng Backend HA đã sẵn sàng. React/Vite được build thành tài nguyên tĩnh, lưu trong S3 private và phân phối qua CloudFront. Cùng một tên miền website xử lý cả giao diện và `/api/*`, nhờ đó cookie xác thực hoạt động nhất quán và Frontend không phụ thuộc trực tiếp vào DNS của ALB.

Kết quả cần đạt:

* Frontend production sử dụng `VITE_API_BASE_URL=/api`.
* Bucket `learnsphere-fe-2` không công khai và chỉ CloudFront OAC được đọc.
* CloudFront cache lâu cho asset có hash nhưng không cache API và `index.html`.
* Route SPA được rewrite về `/index.html` khi người dùng refresh trực tiếp.
* `www.learnspherev2.id.vn` có HTTPS hợp lệ và phân phối website qua CloudFront.
* `origin.learnspherev2.id.vn` có HTTPS hợp lệ và chỉ làm origin Backend.

#### Thành phần production

| Thành phần | Cấu hình |
| --- | --- |
| Frontend | React, TypeScript, Vite |
| Frontend bucket | `learnsphere-fe-2` |
| CDN | CloudFront distribution `E3OLXXMB09Q0AJ` |
| Website | `https://www.learnspherev2.id.vn` |
| Backend origin | `https://origin.learnspherev2.id.vn` |
| TLS cho CloudFront | ACM tại `us-east-1` |
| TLS cho ALB | ACM tại `ap-southeast-1` |
| DNS | TenTen CNAME |

#### Các bước triển khai

1. [Build Frontend và phát hành lên S3](5.6.1-frontend-build-s3/)
2. [Cấu hình CloudFront origins, behaviors và SPA routing](5.6.2-cloudfront-routing/)
3. [Cấu hình ACM và DNS cho tên miền](5.6.3-acm-dns/)
4. [Kiểm tra website production](5.6.4-production-validation/)

#### Luồng request production

```text
Trình duyệt
→ https://www.learnspherev2.id.vn
→ CloudFront
   ├─ /*       → S3 Frontend qua OAC
   └─ /api/*   → HTTPS ALB origin
                  → Target Group
                  → EC2 Backend healthy
```

Hai behavior dùng chính sách cache khác nhau. Asset tĩnh được tối ưu tại edge, trong khi API luôn được chuyển về Backend với cookie và dữ liệu request cần thiết. Thiết kế này giữ giao diện nhanh nhưng không làm sai dữ liệu động.

#### Tiêu chí hoàn thành

Mục 5.6 hoàn tất khi website truy cập được bằng HTTPS trên custom domain, refresh các route SPA không trả 404, request `/api/*` đến đúng ALB, bucket Frontend vẫn private và bản phát hành mới xuất hiện sau CloudFront invalidation.
