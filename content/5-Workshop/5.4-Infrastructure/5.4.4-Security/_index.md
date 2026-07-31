---
title: "Security Groups and infrastructure validation"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

#### Objective

LearnSphere Security Groups implement the following trust chain:

```text
Internet / CloudFront
→ HTTPS:443
→ LearnSphere-ALB-SG
→ TCP:5000
→ LearnSphere-Backend-SG
→ Backend EC2
```

Users cannot connect directly to Backend port 5000. Backend instances have no public IP address and expose no SSH port.

#### 1. ALB Security Group

Create `LearnSphere-ALB-SG` (`sg-0f2fe594908268fc3`) in `LearnSphere-Prod-vpc`.

Current inbound rule:

| Type | Protocol | Port | Source | Purpose |
| --- | --- | --- | --- | --- |
| HTTPS | TCP | 443 | Internet | Accept CloudFront traffic and health validation |

The ALB listener uses the ACM certificate for `origin.learnspherev2.id.vn`. Port 80 is unnecessary unless HTTP-to-HTTPS redirection is explicitly required.

After pipeline health checks are routed through CloudFront, the ALB rule can be restricted further with the AWS-managed CloudFront origin-facing prefix list.

#### 2. Backend Security Group

Create `LearnSphere-Backend-SG` (`sg-0cae310ddce032bbf`) in the same VPC.

Inbound:

| Type | Protocol | Port | Source |
| --- | --- | --- | --- |
| Custom TCP | TCP | 5000 | `sg-0f2fe594908268fc3` |

The source is the ALB Security Group ID, not `0.0.0.0/0` and not an ALB public IP. The rule remains valid when ALB nodes change addresses.

Do not add inbound SSH. Systems Manager Agent uses outbound HTTPS for management.

#### 3. Outbound dependencies

The Backend requires outbound access to:

* Read Parameter Store and communicate with Systems Manager.
* Authenticate to and pull images from ECR.
* Send logs to CloudWatch Logs.
* Access S3 Media through the S3 Gateway Endpoint.
* Connect to MongoDB Atlas over TLS.
* Call Groq and email services over HTTPS.

Private route tables select either the S3 Endpoint or NAT Gateway path. Security Groups are stateful, so response traffic for established connections is automatically allowed.

#### 4. Protect database and object storage

MongoDB Atlas allowlists only:

```text
54.179.11.158/32
52.221.42.74/32
```

Production must not use `0.0.0.0/0`. Database credentials remain in `/learnsphere/prod/backend-env`, not in Security Group descriptions, User Data, or GitHub.

Both S3 buckets retain Block Public Access. CloudFront OAC is the only reader for S3 Frontend, while S3 Media is reached through the EC2 role or time-limited presigned URLs.

#### 5. Validate access paths

| Test | Expected result |
| --- | --- |
| `https://www.learnspherev2.id.vn` | HTTP 200 and the Frontend loads |
| `https://origin.learnspherev2.id.vn/health/ready` | HTTP 200 when Target Group is healthy |
| Backend public IP | None |
| Direct Internet connection to `<private-ip>:5000` | Impossible |
| Media object without a presigned URL | AccessDenied |
| SSM Managed nodes | Both EC2 instances Online |
| MongoDB Atlas allowlist | Only the two production NAT Elastic IPs |

```powershell
curl.exe -i https://origin.learnspherev2.id.vn/health/ready
```

Expected response:

```json
{"status":"ready","database":"connected"}
```

#### 6. Checklist before Section 5.5

* Both Security Groups belong to the production VPC.
* Backend port 5000 accepts only the ALB Security Group.
* No inbound SSH rule exists.
* Private subnets do not auto-assign public IPv4 addresses.
* NAT Elastic IPs match the Atlas allowlist.
* The EC2 instance role has required permissions without AdministratorAccess.
* S3 buckets are private.
* SecureString values do not appear in screenshots or logs.

#### Network security results

![ALB Security Group inbound rule](/images/learnsphere-alb-security-group.png)

*Figure 5.16. `LearnSphere-ALB-SG` belongs to the production VPC and contains a single inbound HTTPS rule on port 443. The ALB does not expose application port 5000.*

![Backend Security Group inbound rule](/images/learnsphere-backend-security-group.png)

*Figure 5.17. `LearnSphere-Backend-SG` accepts TCP port 5000 only from `LearnSphere-ALB-SG` (`sg-0f2fe594908268fc3`) instead of exposing the Backend port to the Internet.*

![Private Backend EC2 network information](/images/learnsphere-private-ec2-network.png)

*Figure 5.18. The Backend EC2 instance resides in a private subnet in `ap-southeast-1b`. It has private IPv4 address `10.20.153.245` but no public IPv4 address, public DNS name, or public IPv6 address.*

These controls reinforce one another: the ALB is the only HTTPS entry point, the Backend trusts only the ALB Security Group, and the EC2 instance has no public address for direct Internet access. Operators manage instances through Systems Manager rather than SSH.
