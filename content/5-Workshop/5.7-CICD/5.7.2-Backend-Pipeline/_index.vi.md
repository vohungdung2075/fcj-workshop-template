---
title: "Pipeline phát hành Backend"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.7.2. </b> "
---

#### 1. Kiểm tra source

Backend job sử dụng Node.js 24:

```bash
cd LearnSphere_BE
npm ci
npm test
```

Test kiểm tra quiz parser, health endpoint, JSON 404, CORS và validation environment production. Pipeline dừng ngay nếu test thất bại.

#### 2. Build image bất biến

```text
ECR URI: 440893644584.dkr.ecr.ap-southeast-1.amazonaws.com/learnsphere-be-2
Image tag: $GITHUB_SHA
```

Workflow kiểm tra image SHA đã tồn tại hay chưa. Nếu chưa, Docker build và push image; nếu đã tồn tại, pipeline tái sử dụng đúng artifact. Không dùng `latest`, nhờ đó release và rollback đều truy ngược được đến commit.

#### 3. Pre-deployment guard

Trước rollout, script xác nhận:

* SHA có đúng 40 ký tự hexadecimal.
* Image tồn tại trong ECR.
* Không có Instance Refresh khác đang chạy.
* ASG có desired capacity tối thiểu 2.
* Số instance healthy bằng desired capacity.

Nếu trạng thái nền không an toàn, workflow dừng trước khi thay instance.

#### 4. Cập nhật release và Instance Refresh

Pipeline đọc tag cũ, ghi SHA mới vào:

```text
/learnsphere/prod/backend-image-tag
```

Sau đó khởi chạy Instance Refresh:

```text
MinHealthyPercentage = 100
MaxHealthyPercentage = 200
InstanceWarmup        = 300
SkipMatching          = false
```

Với launch-before-terminate, ASG khởi tạo instance mới, đợi target healthy rồi mới loại instance cũ. Workflow chờ tối đa 60 phút và không xem lệnh “start refresh” là triển khai thành công.

#### Bằng chứng triển khai

![Các bước triển khai Backend hoàn tất trong GitHub Actions](/images/learnsphere-backend-deploy-job.png)

*Hình 5.41. Backend job hoàn tất kiểm thử, xác thực AWS bằng OIDC, đăng nhập ECR, build và push Docker image, sau đó rollout qua Auto Scaling Group.*

![Các Docker image bất biến được lưu trong Amazon ECR](/images/learnsphere-cicd-ecr-images.png)

*Hình 5.42. Repository `learnsphere-be-2` lưu các image được gắn tag theo commit SHA, hỗ trợ truy vết release và rollback về artifact trước đó.*
