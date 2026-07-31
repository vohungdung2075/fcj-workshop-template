---
title: "Tự động hóa CI/CD bằng GitHub Actions"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Mục tiêu

LearnSphere sử dụng `.github/workflows/deploy.yml` để kiểm tra và phát hành Backend, sau đó mới phát hành Frontend. Pipeline đăng nhập AWS bằng GitHub OIDC và IAM role ngắn hạn, không lưu Access Key dài hạn trong repository.

Kết quả cần đạt:

* Mỗi commit được kiểm tra trước khi phát hành.
* Docker image Backend được gắn đúng Git commit SHA và lưu tại ECR.
* Backend rollout qua ASG Instance Refresh, duy trì tối thiểu 100% healthy capacity.
* Rollout thất bại tự khôi phục image tag trước đó.
* Frontend chỉ phát hành sau khi Backend thành công.
* S3 cache metadata và CloudFront invalidation được thực hiện tự động.
* Hai lần deploy production không chạy chồng lên nhau.

#### Cấu trúc workflow

```text
push main / workflow_dispatch
→ Backend: test → build → ECR → Parameter Store → Instance Refresh → health
→ Frontend: build → S3 sync → index.html → CloudFront invalidation
```

Workflow dùng `concurrency: learnsphere-production` với `cancel-in-progress: false`. Một bản phát hành đang chạy không bị hủy giữa quá trình thay instance, và bản tiếp theo phải chờ.

#### Các bước triển khai

1. [Thiết lập workflow, OIDC và quyền tối thiểu](5.7.1-workflow-oidc/)
2. [Build và phát hành Backend qua ECR/ASG](5.7.2-backend-pipeline/)
3. [Build và phát hành Frontend qua S3/CloudFront](5.7.3-frontend-pipeline/)
4. [Xác minh release và rollback](5.7.4-validation-rollback/)

#### Tiêu chí hoàn thành

CI/CD hoàn tất khi workflow có hai job màu xanh, ECR chứa image gắn SHA của commit, ASG có đủ instance healthy ở hai AZ, public readiness trả HTTP 200, Frontend mới xuất hiện trên custom domain và CloudFront invalidation hoàn thành.
