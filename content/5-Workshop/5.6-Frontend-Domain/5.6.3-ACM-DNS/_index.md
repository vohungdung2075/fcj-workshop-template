---
title: "ACM, HTTPS, and DNS"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.6.3. </b> "
---

#### 1. CloudFront certificate

CloudFront only accepts an ACM certificate in `us-east-1`. The certificate for `www.learnspherev2.id.vn` is validated through the ACM-provided CNAME and attached to the distribution.

#### 2. ALB certificate

The ALB is in `ap-southeast-1`; therefore, the `origin.learnspherev2.id.vn` certificate is created in the same Region and attached to the HTTPS:443 listener. The listener forwards requests to `LearnSphere-Backend-TG`.

#### 3. TenTen DNS records

```text
www.learnspherev2.id.vn
  CNAME → dzr6s0675pe82.cloudfront.net

origin.learnspherev2.id.vn
  CNAME → LearnSphere-Prod-ALB-1917416022.ap-southeast-1.elb.amazonaws.com
```

In addition to the two traffic records, the `_...acm-validations.aws` CNAME records must remain for automatic certificate renewal. DNS values must not include `https://` or a URL path.

#### 4. TLS boundary

```text
Browser --HTTPS--> CloudFront --HTTPS--> ALB --HTTP:5000--> EC2
```

TLS terminates at CloudFront for the website and at the ALB for the origin. ALB-to-EC2 traffic remains inside the VPC and is allowed only from the ALB security group.

CloudFront uses the certificate in `us-east-1`, while the ALB uses the certificate in `ap-southeast-1`; the certificates are not interchangeable.

#### Deployment evidence

![ACM certificate for the frontend domain issued in us-east-1](/images/learnsphere-acm-cloudfront-certificate.png)

*Figure 5.33. The ACM certificate for `www.learnspherev2.id.vn` in `us-east-1` is `Issued`, with successful domain validation.*

![ACM certificate for the ALB origin issued in ap-southeast-1](/images/learnsphere-acm-alb-certificate.png)

*Figure 5.34. The ACM certificate for `origin.learnspherev2.id.vn` in `ap-southeast-1` is `Issued` and ready for the ALB HTTPS listener.*

![Traffic and ACM validation CNAME records in TenTen DNS](/images/learnsphere-tenten-dns-records.png)

*Figure 5.35. TenTen DNS routes `www` to CloudFront and `origin` to the ALB while retaining both ACM validation CNAMEs for automatic certificate renewal.*
