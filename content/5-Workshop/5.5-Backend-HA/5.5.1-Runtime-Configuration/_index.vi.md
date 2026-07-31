---
title: "Cấu hình runtime trong Parameter Store"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

#### Mục tiêu

Backend instance trong ASG có vòng đời ngắn và có thể được thay thế bất kỳ lúc nào. Vì vậy, cấu hình runtime không được lưu thủ công trên từng EC2 hoặc đóng vào Docker image. LearnSphere sử dụng AWS Systems Manager Parameter Store làm nguồn cấu hình tập trung.

#### 1. Phân tách environment và phiên bản image

| Parameter | Type | Người đọc/ghi |
| --- | --- | --- |
| `/learnsphere/prod/backend-env` | `SecureString` | EC2 đọc bằng `LearnSphereEc2Role2`; quản trị viên cập nhật |
| `/learnsphere/prod/backend-image-tag` | `String` | EC2 đọc; GitHub Actions cập nhật và rollback |

`backend-env` chứa secret và biến runtime. `backend-image-tag` chỉ chứa Git commit SHA 40 ký tự của Docker image đang phát hành.

#### 2. Chuẩn bị environment production

Những nhóm biến quan trọng:

```dotenv
PORT=5000
NODE_ENV=production
TRUST_PROXY=1
MONGODB_URI=<secret>
JWT_SECRET=<secret-at-least-64-characters>
FRONTEND_URL=https://www.learnspherev2.id.vn
AWS_REGION=ap-southeast-1
AWS_S3_BUCKET=learnsphere-media-2
AI_PROVIDER=groq
GROQ_API_KEY=<secret>
GROQ_MODEL=<model>
```

`AUTH_COOKIE_DOMAIN` có thể để trống để sử dụng host-only cookie khi Frontend và API cùng đi qua domain `www.learnspherev2.id.vn`. Các biến Bedrock không tham gia runtime khi `AI_PROVIDER=groq`.

#### 3. Tạo SecureString

Từ máy có AWS CLI và quyền phù hợp:

```bash
aws ssm put-parameter \
  --region ap-southeast-1 \
  --name "/learnsphere/prod/backend-env" \
  --description "LearnSphere production backend environment" \
  --type SecureString \
  --tier Standard \
  --key-id alias/aws/ssm \
  --value file:///home/ec2-user/.env \
  --overwrite
```

AWS managed key `alias/aws/ssm` mã hóa giá trị. Chỉ EC2 instance role và quản trị viên được ủy quyền mới có thể gọi `GetParameter` với `WithDecryption=true`.

#### 4. Tạo image tag parameter

Khởi tạo bằng commit SHA đã tồn tại trong ECR:

```bash
aws ssm put-parameter \
  --region ap-southeast-1 \
  --name "/learnsphere/prod/backend-image-tag" \
  --type String \
  --value "<40-character-git-sha>" \
  --overwrite
```

Pipeline đọc tag cũ trước khi phát hành. Nếu Instance Refresh hoặc public health check thất bại, workflow ghi lại tag cũ và khởi chạy rollback refresh.

#### 5. Kiểm tra mà không làm lộ secret

Chỉ kiểm tra metadata hoặc độ dài:

```bash
aws ssm get-parameter \
  --region ap-southeast-1 \
  --name "/learnsphere/prod/backend-env" \
  --with-decryption \
  --query "length(Parameter.Value)" \
  --output text
```

Kiểm tra image tag:

```bash
aws ssm get-parameter \
  --region ap-southeast-1 \
  --name "/learnsphere/prod/backend-image-tag" \
  --query "Parameter.{Name:Name,Type:Type,Version:Version,Value:Value}" \
  --output table
```

Không hiển thị MongoDB URI, JWT secret, email password hoặc Groq API key trong ảnh, terminal history, GitHub log hay báo cáo.

#### 6. MongoDB Atlas Network Access

Hai NAT Elastic IP production được allowlist:

```text
54.179.11.158/32
52.221.42.74/32
```

Nhờ đó, mọi EC2 mới do ASG tạo vẫn kết nối được Atlas qua NAT Gateway cùng AZ mà không cần cập nhật IP theo từng instance.

#### Bằng chứng triển khai

![Các Parameter production của Backend LearnSphere trong Systems Manager Parameter Store](/images/learnsphere-ssm-backend-parameters.png)

*Hình 5.19. Parameter Store lưu `/learnsphere/prod/backend-env` dưới dạng `SecureString` và `/learnsphere/prod/backend-image-tag` dưới dạng `String`. Ảnh xác nhận cấu hình runtime và image tag triển khai được quản lý tập trung mà không làm lộ giá trị secret đã giải mã.*
