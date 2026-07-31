---
title: "Launch Template và User Data bootstrap"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.5.3. </b> "
---

#### Mục tiêu

Launch Template bảo đảm mọi Backend instance được tạo theo cùng một cấu hình. ASG không clone máy chủ đang chạy; nó tạo instance mới từ template và để User Data tái tạo runtime từ Parameter Store cùng ECR.

#### 1. Cấu hình Launch Template

`LearnSphere-Backend-LT`:

| Thuộc tính | Giá trị |
| --- | --- |
| Launch Template ID | `lt-0f60c47d6fe12d8fc` |
| Default version | `2` |
| AMI | Amazon Linux 2023 |
| AMI ID tại thời điểm triển khai | `ami-01b70d44184a858e8` |
| Instance type | `t3.small` |
| Instance profile | `LearnSphereEc2Role2` |
| Security Group | `LearnSphere-Backend-SG` |
| Public IP | Disabled |
| Key pair | Không yêu cầu |

Không chọn subnet cố định trong Launch Template. ASG quyết định subnet theo danh sách hai private subnet để phân bố instance giữa hai AZ.

#### 2. Nguyên tắc User Data

User Data phải:

* Chạy không cần tương tác.
* Dừng khi lệnh quan trọng thất bại.
* Không chứa secret hard-code.
* Có thể chạy trên một instance hoàn toàn mới.
* Chỉ hoàn tất khi readiness endpoint trả HTTP 200.

Script sử dụng:

```bash
#!/bin/bash
set -euxo pipefail

REGION=ap-southeast-1
REGISTRY=440893644584.dkr.ecr.ap-southeast-1.amazonaws.com
REPOSITORY=learnsphere-be-2

dnf install -y docker
systemctl enable --now docker

aws ssm get-parameter \
  --region "$REGION" \
  --name /learnsphere/prod/backend-env \
  --with-decryption \
  --query Parameter.Value \
  --output text > /home/ec2-user/.env

chmod 600 /home/ec2-user/.env

IMAGE_TAG=$(aws ssm get-parameter \
  --region "$REGION" \
  --name /learnsphere/prod/backend-image-tag \
  --query Parameter.Value \
  --output text)

aws ecr get-login-password --region "$REGION" |
  docker login --username AWS --password-stdin "$REGISTRY"

docker pull "$REGISTRY/$REPOSITORY:$IMAGE_TAG"

docker run -d \
  --name learnsphere-be \
  --restart unless-stopped \
  --env-file /home/ec2-user/.env \
  -p 5000:5000 \
  --log-driver awslogs \
  --log-opt awslogs-region="$REGION" \
  --log-opt awslogs-group=/learnsphere/backend2 \
  --log-opt awslogs-stream="backend-$(hostname)" \
  "$REGISTRY/$REPOSITORY:$IMAGE_TAG"

curl --fail --retry 30 --retry-delay 10 \
  http://127.0.0.1:5000/health/ready
```

Trong AWS Console, nhập script dạng text bình thường. Không chọn tùy chọn “User data has already been base64 encoded” nếu chưa encode script.

#### 3. Trình tự bootstrap

1. Amazon Linux khởi động cloud-init.
2. Docker được cài và bật tự động.
3. EC2 role lấy environment SecureString.
4. File `.env` được đặt quyền `600`.
5. EC2 role lấy image tag production.
6. Docker đăng nhập ECR bằng token tạm thời.
7. Image đúng commit SHA được pull.
8. Container chạy với restart policy `unless-stopped`.
9. Docker gửi stdout/stderr vào log group `/learnsphere/backend2`.
10. Script retry readiness tối đa trong thời gian bootstrap.

#### 4. Vì sao dùng Launch Template version 2

Version 1 được tạo trước khi User Data cài Docker đầy đủ, dẫn đến instance lên trạng thái EC2 healthy nhưng không có container và Target Group unhealthy. Version 2 bổ sung bootstrap hoàn chỉnh, sau đó được đặt làm default trước khi ASG tạo lại hai instance.

Việc tạo version mới thay vì sửa âm thầm giúp truy vết lịch sử cấu hình và cho phép quay lại version trước khi cần.

#### 5. Xác minh instance mới

Qua Systems Manager Run Command:

```bash
sudo systemctl status docker --no-pager
sudo docker ps --filter name=learnsphere-be
curl -fsS http://127.0.0.1:5000/health/ready
```

Kết quả cần có Docker active, container running và response `database=connected`.

#### Bằng chứng triển khai

![Launch Template version 2 của Backend LearnSphere](/images/learnsphere-launch-template-v2.png)

*Hình 5.22. `LearnSphere-Backend-LT` sử dụng version 2 làm mặc định, Amazon Linux AMI `ami-01b70d44184a858e8`, instance type `t3.small` và Backend Security Group `sg-0cae310ddce032bbf`. Toàn bộ User Data được chủ động loại khỏi ảnh vì cấu hình vận hành không được làm lộ dữ liệu nhạy cảm.*
