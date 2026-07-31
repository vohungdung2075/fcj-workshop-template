---
title: "Build Frontend và phát hành lên S3"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.6.1. </b> "
---

#### 1. Chuẩn bị cấu hình build

Frontend chỉ gọi API qua cùng origin:

```dotenv
VITE_API_BASE_URL=/api
```

Cách cấu hình này tránh hard-code ALB hoặc CloudFront domain vào bundle. Trình duyệt gửi request đến `www.learnspherev2.id.vn/api/*`, sau đó CloudFront chọn behavior Backend.

#### 2. Cài dependency và kiểm tra build

```bash
cd LearnSphere_FE
npm ci
npm run build
```

Vite tạo thư mục `dist/` gồm `index.html` và các file CSS, JavaScript, font đã gắn hash. Trước khi upload cần xác nhận build không có lỗi TypeScript và không chứa secret Backend.

#### 3. Đồng bộ theo chính sách cache

Pipeline phát hành theo ba nhóm:

```bash
# Asset có hash: cache dài hạn
aws s3 sync dist/assets/ s3://learnsphere-fe-2/assets/ \
  --cache-control "public,max-age=31536000,immutable"

# Tệp tĩnh còn lại: cache ngắn
aws s3 sync dist/ s3://learnsphere-fe-2/ \
  --exclude "assets/*" --exclude "index.html" \
  --cache-control "public,max-age=3600"

# HTML entry point: luôn kiểm tra phiên bản mới
aws s3 cp dist/index.html s3://learnsphere-fe-2/index.html \
  --cache-control "no-cache,no-store,must-revalidate"
```

`index.html` được upload cuối để không trỏ người dùng đến asset chưa tồn tại. Workflow không dùng `--delete`; object cũ được xử lý bằng lifecycle nhằm tránh xóa asset vẫn đang được một phiên trình duyệt cũ sử dụng.

#### 4. Bảo vệ bucket

Block Public Access được giữ nguyên. Bucket policy chỉ cấp `s3:GetObject` cho CloudFront service principal khi `AWS:SourceArn` khớp distribution `E3OLXXMB09Q0AJ`. Người dùng không truy cập S3 trực tiếp.

#### Bằng chứng triển khai

![Các object Frontend đã được phát hành lên bucket private learnsphere-fe-2](/images/learnsphere-frontend-s3-objects.png)

*Hình 5.28. Bucket `learnsphere-fe-2` chứa `index.html`, thư mục `assets/` và favicon sau khi Frontend được build và phát hành.*

![Metadata cache của index.html trong S3](/images/learnsphere-s3-index-cache-metadata.png)

*Hình 5.29a. `index.html` sử dụng `no-cache, no-store, must-revalidate` để trình duyệt luôn kiểm tra phiên bản Frontend mới.*

![Metadata cache của JavaScript asset có hash trong S3](/images/learnsphere-s3-hashed-asset-cache-metadata.png)

*Hình 5.29b. JavaScript asset có hash sử dụng `public, max-age=31536000, immutable` để tận dụng cache dài hạn an toàn.*
