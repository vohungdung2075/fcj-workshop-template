---
title: "IAM, GitHub OIDC và quyền runtime"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

#### Mục tiêu

LearnSphere tách danh tính triển khai và danh tính runtime thành hai role độc lập:

| Role | Principal | Mục đích |
| --- | --- | --- |
| `LearnSphereGitHubDeployRole2` | GitHub Actions qua OIDC | Build và phát hành ứng dụng |
| `LearnSphereEc2Role2` | Amazon EC2 | Khởi động và vận hành Backend |

Cách tách này ngăn pipeline có quyền đọc toàn bộ secret runtime, đồng thời ngăn EC2 có quyền sửa CloudFront hoặc triển khai Frontend.

#### 1. Tạo GitHub OIDC provider

Trong IAM Console, mở **Identity providers → Add provider** và nhập:

| Trường | Giá trị |
| --- | --- |
| Provider type | OpenID Connect |
| Provider URL | `https://token.actions.githubusercontent.com` |
| Audience | `sts.amazonaws.com` |

OIDC cho phép GitHub Actions đổi token của workflow lấy AWS credential có thời hạn ngắn. Repository không cần lưu `AWS_ACCESS_KEY_ID` hoặc `AWS_SECRET_ACCESS_KEY`.

#### 2. Tạo deployment role

Tạo role `LearnSphereGitHubDeployRole2` với trust policy giới hạn đúng repository, audience và nhánh production:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::440893644584:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:HoiaeKHMT/LearnSphere:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

Điều kiện `sub` rất quan trọng: workflow từ fork, repository khác hoặc nhánh khác không thể assume role này.

#### 3. Cấp quyền triển khai tối thiểu

Deployment role được chia quyền theo từng nhiệm vụ:

| Nhóm quyền | Hành động cần thiết |
| --- | --- |
| ECR | Lấy authorization token, kiểm tra layer, upload layer và push image |
| S3 Frontend | List bucket, upload, đọc và xóa object cũ khi `s3 sync --delete` |
| CloudFront | Tạo và đọc trạng thái invalidation |
| Parameter Store | Đọc và cập nhật `/learnsphere/prod/backend-image-tag` |
| Auto Scaling | Đọc ASG, tạo và theo dõi Instance Refresh |
| EC2/ELB read-only | Đọc instance và trạng thái Target Group phục vụ validation |

Resource ARN được giới hạn tới `learnsphere-be-2`, `learnsphere-fe-2`, CloudFront distribution, Parameter Store path và `LearnSphere-Backend-ASG` khi dịch vụ hỗ trợ resource-level permission.

#### 4. Tạo EC2 instance role

Tạo `LearnSphereEc2Role2`, sau đó gắn role vào instance profile được Launch Template sử dụng. Role runtime cần:

* Đọc và giải mã `/learnsphere/prod/backend-env`.
* Đọc `/learnsphere/prod/backend-image-tag`.
* Lấy ECR authorization token và pull image từ `learnsphere-be-2`.
* Đọc, ghi, xóa object trong `learnsphere-media-2` theo nghiệp vụ.
* Ghi log container vào CloudWatch Logs.
* Kết nối Systems Manager để vận hành instance không cần SSH.

Role không cần quyền đồng bộ S3 Frontend, tạo CloudFront invalidation hoặc thay đổi ASG.

#### 5. Lưu cấu hình trong Parameter Store

Hai parameter được tách riêng:

| Parameter | Type | Nội dung |
| --- | --- | --- |
| `/learnsphere/prod/backend-env` | `SecureString` | Toàn bộ biến môi trường production |
| `/learnsphere/prod/backend-image-tag` | `String` | Git commit SHA đang được phát hành |

Tách image tag khỏi environment giúp pipeline đổi phiên bản Backend mà không phải đọc hoặc ghi lại MongoDB URI, JWT secret, email password và Groq API key.

Ví dụ tạo parameter môi trường:

```bash
aws ssm put-parameter \
  --region ap-southeast-1 \
  --name "/learnsphere/prod/backend-env" \
  --description "LearnSphere production backend environment" \
  --type SecureString \
  --key-id alias/aws/ssm \
  --value file://.env \
  --overwrite
```

#### 6. Xác minh

Kiểm tra role GitHub bằng một workflow có:

```yaml
permissions:
  id-token: write
  contents: read
```

Sau bước `aws-actions/configure-aws-credentials`, chạy:

```bash
aws sts get-caller-identity
```

Kết quả phải trả về assumed role `LearnSphereGitHubDeployRole2`. Trên EC2, lệnh tương tự phải trả về assumed role `LearnSphereEc2Role2`.

#### Kết quả cấu hình IAM

![GitHub OIDC provider của LearnSphere](/images/learnsphere-github-oidc-role.png)

*Hình 5.9. GitHub OIDC provider `token.actions.githubusercontent.com` được cấu hình theo chuẩn OpenID Connect với audience `sts.amazonaws.com`. Trust policy giới hạn repository và nhánh triển khai đã được trình bày trong đoạn mã phía trên.*

![Hai IAM role của LearnSphere](/images/learnsphere-iam-roles.png)

*Hình 5.10. Hai IAM role được tách theo chức năng: `LearnSphereGitHubDeployRole2` dành cho quy trình CI/CD và `LearnSphereEc2Role2` dành cho các Backend EC2 instance.*

Kết quả trên xác nhận LearnSphere không dùng chung một identity cho cả deployment và runtime. Mỗi role chỉ được cấp nhóm quyền cần thiết cho nhiệm vụ của mình, giúp giảm phạm vi ảnh hưởng nếu một thành phần gặp sự cố bảo mật.
