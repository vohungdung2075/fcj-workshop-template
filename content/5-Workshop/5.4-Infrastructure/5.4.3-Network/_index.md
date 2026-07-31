---
title: "Multi-AZ VPC network"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

#### Objective

The production network is designed so that:

* The Application Load Balancer accepts HTTPS in the public tier.
* Backend EC2 instances run only in private subnets without public IPv4 addresses.
* Each Availability Zone has independent outbound connectivity through its local NAT Gateway.
* Backend-to-S3 traffic uses a Gateway Endpoint.
* The service remains available when one Backend instance or one AZ target fails.

#### 1. Create the VPC

Create `LearnSphere-Prod-vpc` with CIDR `10.20.0.0/16`, then enable:

* DNS resolution.
* DNS hostnames.

DNS is required for EC2 instances to resolve ECR, SSM, MongoDB Atlas, Groq, and AWS endpoints.

#### 2. Create four subnets

| Tier | Availability Zone | Subnet ID | Resources |
| --- | --- | --- | --- |
| Public 1 | `ap-southeast-1a` | `subnet-0e569ab8d7b6bc218` | ALB and NAT Gateway 1a |
| Public 2 | `ap-southeast-1b` | `subnet-047a2b20ba02c7fec` | ALB and NAT Gateway 1b |
| Private 1 | `ap-southeast-1a` | `subnet-04724edeb47832763` | Backend EC2 |
| Private 2 | `ap-southeast-1b` | `subnet-0ccb5a5e29560fa75` | Backend EC2 |

Public subnets may auto-assign public IPv4 addresses when required. Private subnets keep this option disabled.

#### 3. Internet Gateway and public route table

Create and attach Internet Gateway `igw-09089b621ff5ed1c9`. Associate `LearnSphere-Prod-rtb-public` (`rtb-0322485778e8cc7d9`) with both public subnets:

| Destination | Target |
| --- | --- |
| `10.20.0.0/16` | `local` |
| `0.0.0.0/0` | `igw-09089b621ff5ed1c9` |

An Internet Gateway does not use a Security Group. Route tables provide Internet routing, while the ALB Security Group controls accepted traffic.

#### 4. Deploy one NAT Gateway per AZ

Create one public NAT Gateway in each public subnet:

| NAT Gateway | AZ | Elastic IP |
| --- | --- | --- |
| `nat-05cd24211262862a9` | `ap-southeast-1a` | `54.179.11.158` |
| `nat-0cdcc979207bf10d7` | `ap-southeast-1b` | `52.221.42.74` |

Both Elastic IPs are allowlisted in MongoDB Atlas Network Access. Auto Scaling may replace the private instance addresses, but Atlas continues to observe the stable NAT egress addresses.

#### 5. Configure private route tables

Each private subnet has a dedicated route table:

| Private route table | Subnet | Default route |
| --- | --- | --- |
| `rtb-01b717b7bd49e1fa9` | Private 1a | `0.0.0.0/0 → nat-05cd24211262862a9` |
| `rtb-052af44704ed1ae20` | Private 1b | `0.0.0.0/0 → nat-0cdcc979207bf10d7` |

Routing each private subnet through its local NAT avoids a cross-AZ dependency and unnecessary cross-AZ data transfer.

#### 6. Add the S3 Gateway Endpoint

Create S3 endpoint `vpce-010ad76d21bca4533` and associate it with both private route tables. AWS adds a route through managed prefix list `pl-6fa54006`.

This design:

* Keeps S3 object traffic on the AWS network.
* Reduces NAT Gateway data processing.
* Preserves IAM and bucket policy authorization for S3 Media.

ECR image pulls still require ECR API, ECR DKR, and image layers stored in S3. The current architecture uses NAT for services that do not have dedicated Interface VPC Endpoints.

#### 7. Network flows

Inbound:

```text
User
→ CloudFront
→ ALB HTTPS:443 across two public subnets
→ Target Group HTTP:5000
→ Backend EC2 across two private subnets
```

Outbound:

```text
Backend EC2 AZ 1a → NAT Gateway 1a → MongoDB Atlas / Groq / email
Backend EC2 AZ 1b → NAT Gateway 1b → MongoDB Atlas / Groq / email
Backend EC2 in both AZs → S3 Gateway Endpoint → Amazon S3
```

A NAT Gateway only supports connections initiated by private resources; it does not allow unsolicited inbound Internet connections to EC2.

#### 8. Validate routing

* The VPC Resource Map shows four subnets across two AZs.
* The public route table is associated with both public subnets and routes to the IGW.
* Each private route table points to the local NAT Gateway.
* Both private route tables include the S3 prefix-list route.
* Private EC2 instances have no public IPv4 address but can resolve DNS and reach ECR, SSM, Atlas, and Groq.

#### Multi-AZ routing results

![LearnSphere public route table](/images/learnsphere-public-route-table.png)

*Figure 5.13. `LearnSphere-Prod-rtb-public` is explicitly associated with two public subnets and routes `0.0.0.0/0` to Internet Gateway `igw-09089b621ff5ed1c9`.*

![Private route table in Availability Zone 1a](/images/learnsphere-private-route-table-1a.png)

*Figure 5.14a. The private route table in `ap-southeast-1a` sends Internet egress through NAT Gateway `nat-05cd24211262862a9` and reaches Amazon S3 through Gateway Endpoint `vpce-010ad76d21bca4533`.*

![Private route table in Availability Zone 1b](/images/learnsphere-private-route-table-1b.png)

*Figure 5.14b. The private route table in `ap-southeast-1b` uses local NAT Gateway `nat-0cdcc979207bf10d7` and shares the S3 Gateway Endpoint with private subnet 1a.*

![LearnSphere NAT Gateways](/images/learnsphere-nat-gateways.png)

*Figure 5.15. Both production NAT Gateways use `Public` connectivity, have `Zonal` scope, and are `Available`, providing independent outbound paths for the two Availability Zones.*

This evidence confirms that the private subnets do not share one NAT Gateway. If one NAT Gateway or one Availability Zone network path fails, the Backend in the other Availability Zone retains an independent egress path to MongoDB Atlas, Groq, and other external services.
