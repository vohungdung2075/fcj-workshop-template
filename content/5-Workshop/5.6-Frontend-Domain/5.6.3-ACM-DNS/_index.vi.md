---
title: "ACM, HTTPS và DNS"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.6.3. </b> "
---

#### 1. Chứng chỉ cho CloudFront

CloudFront chỉ chấp nhận ACM certificate tại `us-east-1`. Certificate cho `www.learnspherev2.id.vn` được xác thực bằng CNAME do ACM cung cấp và gắn vào distribution.

#### 2. Chứng chỉ cho ALB

ALB nằm tại `ap-southeast-1`, do đó certificate `origin.learnspherev2.id.vn` được tạo trong cùng Region và gắn vào listener HTTPS:443. Listener forward request đến `LearnSphere-Backend-TG`.

#### 3. Bản ghi DNS tại TenTen

```text
www.learnspherev2.id.vn
  CNAME → dzr6s0675pe82.cloudfront.net

origin.learnspherev2.id.vn
  CNAME → LearnSphere-Prod-ALB-1917416022.ap-southeast-1.elb.amazonaws.com
```

Ngoài hai CNAME phục vụ traffic, các CNAME `_...acm-validations.aws` phải được giữ lại để ACM tự động gia hạn certificate. Không đưa giao thức `https://` hoặc đường dẫn vào giá trị DNS.

#### 4. Ranh giới TLS

```text
Browser --HTTPS--> CloudFront --HTTPS--> ALB --HTTP:5000--> EC2
```

TLS được chấm dứt tại CloudFront đối với website và tại ALB đối với origin. Traffic từ ALB đến EC2 nằm trong VPC, chỉ được Backend security group cho phép từ ALB security group.

CloudFront dùng certificate ở `us-east-1`, còn ALB dùng certificate cùng Region `ap-southeast-1`; hai certificate không thể thay thế vị trí cho nhau.

#### Bằng chứng triển khai

![Chứng chỉ ACM cho tên miền Frontend được phát hành tại us-east-1](/images/learnsphere-acm-cloudfront-certificate.png)

*Hình 5.33. Chứng chỉ ACM cho `www.learnspherev2.id.vn` tại `us-east-1` đã ở trạng thái `Issued` và xác thực tên miền thành công.*

![Chứng chỉ ACM cho ALB origin được phát hành tại ap-southeast-1](/images/learnsphere-acm-alb-certificate.png)

*Hình 5.34. Chứng chỉ ACM cho `origin.learnspherev2.id.vn` tại `ap-southeast-1` đã ở trạng thái `Issued` và sẵn sàng cho listener HTTPS của ALB.*

![Các bản ghi CNAME traffic và ACM validation tại TenTen](/images/learnsphere-tenten-dns-records.png)

*Hình 5.35. DNS TenTen định tuyến `www` đến CloudFront, `origin` đến ALB và duy trì hai CNAME validation để ACM tự động gia hạn chứng chỉ.*
