---
title: "Pipeline phát hành Frontend"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.7.3. </b> "
---

#### 1. Phụ thuộc vào Backend

```yaml
deploy-frontend:
  needs: deploy-backend
```

Frontend chỉ chạy khi Backend đã test, rollout và vượt qua public health check. Điều này tránh phát hành UI gọi API chưa sẵn sàng.

#### 2. Kiểm tra API base URL

Pipeline chấp nhận:

* `/api` cho same-origin production; hoặc
* HTTPS URL hợp lệ khi triển khai môi trường khác.

Giá trị HTTP hoặc chuỗi không hợp lệ làm job thất bại trước build.

#### 3. Build và sync S3

```bash
cd LearnSphere_FE
npm ci
npm run build
```

Assets có hash được sync với cache một năm và `immutable`; file khác cache một giờ; `index.html` được upload cuối với `no-cache,no-store,must-revalidate`.

Pipeline không đưa AWS credential vào Vite environment. OIDC chỉ tồn tại trong runner để gọi S3 và CloudFront.

#### 4. Invalidate CloudFront

Sau khi upload thành công:

```bash
aws cloudfront create-invalidation \
  --distribution-id "$CLOUDFRONT_FE_DISTRIBUTION_ID" \
  --paths "/*"
```

Job chỉ hoàn tất sau khi AWS chấp nhận invalidation. Người dùng sau đó nhận entry point mới, còn tên asset theo hash bảo đảm tránh xung đột cache.

#### Bằng chứng triển khai

![Các bước triển khai Frontend hoàn tất trong GitHub Actions](/images/learnsphere-frontend-deploy-job.png)

*Hình 5.43. Frontend job hoàn tất cài dependency, build production bundle, đồng bộ nội dung lên S3 và tạo CloudFront invalidation.*

![Các CloudFront invalidation sau khi pipeline phát hành Frontend](/images/learnsphere-cicd-cloudfront-invalidations.png)

*Hình 5.44. CloudFront ghi nhận các invalidation ở trạng thái `Completed`, xác nhận edge cache đã được làm mới sau các lần phát hành Frontend.*
