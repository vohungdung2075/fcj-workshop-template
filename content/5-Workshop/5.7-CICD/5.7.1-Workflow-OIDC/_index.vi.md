---
title: "Workflow, OIDC và quyền triển khai"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.7.1. </b> "
---

#### 1. Trigger và quyền GitHub

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

concurrency:
  group: learnsphere-production
  cancel-in-progress: false
```

`contents: read` đủ để checkout source. `id-token: write` cho phép runner yêu cầu OIDC token; đây không phải quyền ghi source code.

#### 2. Assume role bằng OIDC

```yaml
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: ${{ secrets.AWS_GITHUB_ROLE_ARN }}
    aws-region: ap-southeast-1
```

Trust policy của `LearnSphereGitHubDeployRole2` giới hạn:

* Provider: `token.actions.githubusercontent.com`.
* Audience: `sts.amazonaws.com`.
* Subject: repository LearnSphere và branch `main`.

AWS STS cấp credential tạm thời cho từng job. Khi job kết thúc credential hết hiệu lực, giảm rủi ro so với lưu `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY`.

#### 3. Phạm vi quyền deploy

Role chỉ cần các nhóm quyền:

* Push/pull metadata của repository `learnsphere-be-2`.
* Đọc/ghi `/learnsphere/prod/backend-image-tag`.
* Đọc ASG và thực hiện Instance Refresh.
* Đồng bộ object vào `learnsphere-fe-2`.
* Tạo invalidation cho distribution được chỉ định.

EC2 dùng role riêng `LearnSphereEc2Role2` để đọc environment, pull image, truy cập S3 Media và ghi CloudWatch Logs. Không dùng chung deployment role với runtime role.

#### 4. Biến và secret GitHub

| Tên | Loại | Ý nghĩa |
| --- | --- | --- |
| `AWS_GITHUB_ROLE_ARN` | Secret | Role được GitHub OIDC assume |
| `S3_FE_BUCKET` | Secret/Variable | Bucket Frontend |
| `CLOUDFRONT_FE_DISTRIBUTION_ID` | Secret/Variable | Distribution cần invalidate |
| `VITE_API_BASE_URL` | Secret/Variable | `/api` trong production |

Chỉ chụp tên secret/variable; GitHub không hiển thị và báo cáo không được chứa giá trị nhạy cảm.

#### Bằng chứng triển khai

![Workflow GitHub Actions triển khai LearnSphere lên AWS](/images/learnsphere-github-actions-workflow.png)

*Hình 5.39. Workflow production được kích hoạt khi push vào `main` hoặc chạy thủ công, giới hạn deploy đồng thời, cấp quyền OIDC và cấu hình triển khai Backend qua Auto Scaling Group.*

![Danh sách Repository Secrets phục vụ workflow production](/images/learnsphere-github-repository-secrets.png)

*Hình 5.40. GitHub chỉ hiển thị tên bốn repository secret cần thiết cho OIDC, S3, CloudFront và cấu hình API; các giá trị bí mật không xuất hiện trong giao diện hoặc báo cáo.*
