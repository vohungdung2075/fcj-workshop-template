---
title: "Mạng VPC Multi-AZ"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

#### Mục tiêu

Mạng production được thiết kế để:

* Application Load Balancer nhận HTTPS tại lớp public.
* Backend EC2 chỉ chạy trong private subnet và không có public IPv4.
* Mỗi Availability Zone có đường egress riêng qua NAT Gateway cùng AZ.
* Traffic từ Backend tới S3 đi qua Gateway Endpoint.
* Hệ thống tiếp tục phục vụ khi một Backend instance hoặc một AZ target gặp lỗi.

#### 1. Tạo VPC

Tạo `LearnSphere-Prod-vpc` với CIDR `10.20.0.0/16`, sau đó bật:

* DNS resolution.
* DNS hostnames.

Hai tùy chọn DNS cần thiết để EC2 phân giải ECR, SSM, MongoDB Atlas, Groq và các AWS endpoint.

#### 2. Tạo bốn subnet

| Lớp | Availability Zone | Subnet ID | Tài nguyên |
| --- | --- | --- | --- |
| Public 1 | `ap-southeast-1a` | `subnet-0e569ab8d7b6bc218` | ALB, NAT Gateway 1a |
| Public 2 | `ap-southeast-1b` | `subnet-047a2b20ba02c7fec` | ALB, NAT Gateway 1b |
| Private 1 | `ap-southeast-1a` | `subnet-04724edeb47832763` | Backend EC2 |
| Private 2 | `ap-southeast-1b` | `subnet-0ccb5a5e29560fa75` | Backend EC2 |

Public subnet bật auto-assign public IPv4 khi cần cho tài nguyên public. Private subnet tắt chức năng này.

#### 3. Internet Gateway và public route table

Tạo và gắn Internet Gateway `igw-09089b621ff5ed1c9` vào VPC. Route table `LearnSphere-Prod-rtb-public` (`rtb-0322485778e8cc7d9`) được liên kết với hai public subnet:

| Destination | Target |
| --- | --- |
| `10.20.0.0/16` | `local` |
| `0.0.0.0/0` | `igw-09089b621ff5ed1c9` |

Internet Gateway không cần nối trực tiếp bằng Security Group. Route table quyết định public subnet có đường Internet, còn Security Group kiểm soát traffic tới ALB.

#### 4. NAT Gateway theo từng AZ

Tạo một public NAT Gateway trong mỗi public subnet:

| NAT Gateway | AZ | Elastic IP |
| --- | --- | --- |
| `nat-05cd24211262862a9` | `ap-southeast-1a` | `54.179.11.158` |
| `nat-0cdcc979207bf10d7` | `ap-southeast-1b` | `52.221.42.74` |

Hai Elastic IP được allowlist trong MongoDB Atlas Network Access. Khi ASG thay EC2, địa chỉ private của instance thay đổi nhưng địa chỉ egress nhìn từ Atlas vẫn là hai NAT Elastic IP.

#### 5. Private route tables

Mỗi private subnet có route table riêng:

| Private route table | Subnet | Default route |
| --- | --- | --- |
| `rtb-01b717b7bd49e1fa9` | Private 1a | `0.0.0.0/0 → nat-05cd24211262862a9` |
| `rtb-052af44704ed1ae20` | Private 1b | `0.0.0.0/0 → nat-0cdcc979207bf10d7` |

Không trỏ cả hai private subnet vào cùng một NAT Gateway. Cấu hình NAT cùng AZ tránh phụ thuộc chéo Availability Zone và giảm cross-AZ data transfer.

#### 6. S3 Gateway Endpoint

Tạo endpoint `vpce-010ad76d21bca4533` cho service S3 và liên kết với hai private route table. AWS thêm route sử dụng managed prefix list `pl-6fa54006`.

Kết quả:

* EC2 pull hoặc truy cập object S3 mà không đi qua Internet.
* Giảm data processing qua NAT Gateway.
* S3 Media vẫn được kiểm soát bằng IAM và bucket policy.

ECR image pull vẫn cần truy cập ECR API, ECR DKR và lớp image lưu trên S3. Kiến trúc hiện dùng NAT Gateway cho các endpoint chưa có Interface VPC Endpoint.

#### 7. Luồng mạng

Luồng vào:

```text
User
→ CloudFront
→ ALB HTTPS:443 trong hai public subnet
→ Target Group HTTP:5000
→ Backend EC2 trong hai private subnet
```

Luồng ra:

```text
Backend EC2 AZ 1a → NAT Gateway 1a → MongoDB Atlas / Groq / email
Backend EC2 AZ 1b → NAT Gateway 1b → MongoDB Atlas / Groq / email
Backend EC2 ở cả hai AZ → S3 Gateway Endpoint → Amazon S3
```

NAT Gateway chỉ hỗ trợ kết nối do private resource khởi tạo; nó không cho Internet chủ động kết nối vào EC2.

#### 8. Xác minh

* VPC Resource Map hiển thị bốn subnet trên hai AZ.
* Public route table liên kết đúng hai public subnet và có route tới IGW.
* Mỗi private route table trỏ tới NAT Gateway cùng AZ.
* Hai private route table đều có route S3 prefix list tới endpoint.
* EC2 private không có public IPv4 nhưng vẫn phân giải DNS và truy cập ECR, SSM, Atlas, Groq.

#### Kết quả cấu hình định tuyến Multi-AZ

![Public route table của LearnSphere](/images/learnsphere-public-route-table.png)

*Hình 5.13. Public route table `LearnSphere-Prod-rtb-public` được liên kết tường minh với hai public subnet và có default route `0.0.0.0/0` tới Internet Gateway `igw-09089b621ff5ed1c9`.*

![Private route table tại Availability Zone 1a](/images/learnsphere-private-route-table-1a.png)

*Hình 5.14a. Private route table tại `ap-southeast-1a` đưa Internet egress qua NAT Gateway `nat-05cd24211262862a9` và truy cập Amazon S3 qua Gateway Endpoint `vpce-010ad76d21bca4533`.*

![Private route table tại Availability Zone 1b](/images/learnsphere-private-route-table-1b.png)

*Hình 5.14b. Private route table tại `ap-southeast-1b` sử dụng NAT Gateway `nat-0cdcc979207bf10d7` cùng AZ và dùng chung S3 Gateway Endpoint với private subnet 1a.*

![Hai NAT Gateway của LearnSphere](/images/learnsphere-nat-gateways.png)

*Hình 5.15. Hai NAT Gateway production đều có connectivity type `Public`, phạm vi `Zonal` và trạng thái `Available`, cung cấp egress độc lập cho hai Availability Zone.*

Các bằng chứng trên xác nhận hai private subnet không dùng chung một NAT Gateway. Nếu một NAT Gateway hoặc đường mạng trong một Availability Zone gặp lỗi, Backend thuộc Availability Zone còn lại vẫn có đường egress riêng để kết nối MongoDB Atlas, Groq và các dịch vụ bên ngoài.
