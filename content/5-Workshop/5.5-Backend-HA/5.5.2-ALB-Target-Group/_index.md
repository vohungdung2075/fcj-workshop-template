---
title: "Target Group and Application Load Balancer"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---

#### Objective

The Application Load Balancer provides a stable HTTPS entry point for multiple replaceable Backend instances. The Target Group evaluates readiness and routes requests only to instances whose application and database are available.

#### 1. Create the Target Group

Create `LearnSphere-Backend-TG`:

| Attribute | Value |
| --- | --- |
| Target type | Instance |
| Protocol | HTTP |
| Port | 5000 |
| Protocol version | HTTP1 |
| VPC | `vpc-0d59e8bb67e90a0da` |

Health check:

| Attribute | Value |
| --- | --- |
| Protocol | HTTP |
| Port | Traffic port |
| Path | `/health/ready` |
| Success code | `200` |

The readiness endpoint returns HTTP 200 only when Node.js is running and MongoDB is connected:

```json
{
  "status": "ready",
  "database": "connected"
}
```

If the database disconnects, the target does not report ready. The ALB removes it from rotation rather than forwarding business requests to a Backend that cannot complete them.

#### 2. Create the Application Load Balancer

Create `LearnSphere-Prod-ALB`:

| Attribute | Value |
| --- | --- |
| Scheme | Internet-facing |
| IP address type | IPv4 |
| VPC | `LearnSphere-Prod-vpc` |
| Subnets | Public 1a and Public 1b |
| Security Group | `LearnSphere-ALB-SG` |

The ALB spans two public subnets so that it has nodes in both Availability Zones. Backend EC2 instances remain in private subnets.

#### 3. Configure the HTTPS listener

Production listener:

```text
Protocol: HTTPS
Port: 443
Certificate: origin.learnspherev2.id.vn
Default action: Forward to LearnSphere-Backend-TG
```

The ALB certificate must be created in `ap-southeast-1`, the same Region as the ALB. TenTen DNS contains:

```text
origin.learnspherev2.id.vn
→ LearnSphere-Prod-ALB-1917416022.ap-southeast-1.elb.amazonaws.com
```

The ALB terminates TLS and forwards HTTP on port 5000 inside the VPC. The Backend Security Group accepts only the ALB Security Group as its source.

#### 4. Attach the Target Group to the ASG

Do not maintain long-lived manual target registrations. Attach `LearnSphere-Backend-ASG` directly to the Target Group:

* New instances register automatically.
* Terminated instances deregister automatically.
* The ALB uses only healthy targets.
* The ASG can replace instances based on ELB health.

#### 5. Validate the ingress layer

Before the ASG provides instances, the ALB may return HTTP 503 because the Target Group has no healthy target. After both Backends bootstrap:

```powershell
curl.exe -i https://origin.learnspherev2.id.vn/health/ready
```

The response must be HTTP 200. The ALB listener must forward 100% of traffic to `LearnSphere-Backend-TG`.

#### Deployment evidence

![LearnSphere ALB network mapping across two public subnets](/images/learnsphere-alb-network-mapping.png)

*Figure 5.20. `LearnSphere-Prod-ALB` is mapped to public subnets in `ap-southeast-1a` and `ap-southeast-1b` inside VPC `10.20.0.0/16`, providing a redundant ingress path across two Availability Zones.*

![LearnSphere ALB HTTPS listener and Target Group forwarding rule](/images/learnsphere-alb-https-listener.png)

*Figure 5.21. The ALB listens on HTTPS port 443, uses the certificate for `origin.learnspherev2.id.vn`, and forwards 100% of matching traffic to `LearnSphere-Backend-TG`.*
