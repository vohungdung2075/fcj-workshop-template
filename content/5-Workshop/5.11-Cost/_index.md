---
title: "Cost Analysis"
date: 2026-07-31
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

#### Analysis objective

This section estimates the operating cost of the **LearnSphere As-Built** architecture in the Singapore Region (`ap-southeast-1`). The architecture includes two `t3.small` EC2 instances in an Auto Scaling Group, two NAT Gateways, one Application Load Balancer, Amazon S3, Amazon ECR, Amazon CloudFront, CloudWatch, and related management services.




#### Input assumptions

| Assumption group | Value |
| --- | --- |
| Region | Asia Pacific (Singapore) – `ap-southeast-1` |
| Active runtime | 240 hours/month |
| Backend | 2 Linux On-Demand `t3.small` EC2 instances |
| High Availability | 2 Availability Zones, 2 private subnets, 2 NAT Gateways |
| Load balancing | 1 internet-facing Application Load Balancer |
| EC2 storage | 2 gp3 EBS volumes, 8 GB each |
| NAT data processing | Approximately 10 GB/month |
| S3 storage | Approximately 11 GB for frontend and media |
| ECR storage | Approximately 1 GB of retained images |
| CloudFront | Free plan, workshop traffic below allowance |
| MongoDB Atlas and Groq | Free plans during the workshop phase |

#### Estimated monthly cost

| Service / Resource | Workshop calculation | Estimated cost (USD/month) |
| --- | --- | ---: |
| Amazon EC2 | 2 × `t3.small` × 240 hours × USD 0.0268 | 12.86 |
| Amazon EBS gp3 | 2 volumes × 8 GB; storage retained continuously | 1.54 |
| NAT Gateway – hourly charge | 2 NAT Gateways × 240 hours × USD 0.059 | 28.32 |
| NAT Gateway – data processing | 10 GB × USD 0.059 | 0.59 |
| Application Load Balancer | 1 ALB × 240 hours × USD 0.0252 | 6.05 |
| ALB Capacity Unit | Average workshop load of approximately 0.25 LCU | 0.48 |
| Public IPv4 | 2 NAT Gateway IPv4 addresses × 240 hours | 2.40 |
| Amazon S3 Standard | Approximately 11 GB of frontend and media data | 0.28 |
| Amazon S3 Requests | Low-volume PUT and GET requests | 0.05 |
| Amazon ECR | Approximately 1 GB of Docker images | 0.10 |
| Amazon CloudWatch | Low-volume log ingestion, storage, and metrics | 0.75 |
| Amazon CloudFront | Free plan, below plan allowance | 0.00 |
| ACM, IAM, SSM Parameter Store, and S3 Gateway Endpoint | No base resource charge under the current configuration | 0.00 |
| **Infrastructure subtotal** |  | **53.42** |
| **Approximately 5% contingency** | Traffic, requests, and logs beyond the assumptions | **2.67** |
| **ESTIMATED TOTAL** | Controlled workshop operating scenario | **USD 56.09/month** |

The target therefore remains within **USD 50–60 per month**. This estimate excludes taxes and may change with exchange rates, actual resource runtime, network traffic, and AWS price updates.

#### Cost structure analysis

**1. NAT Gateway is the largest cost component**

The two NAT Gateways are estimated at USD 28.91, including hourly charges and data processing, which represents more than half of the infrastructure subtotal. One NAT Gateway is used per Availability Zone to avoid cross-AZ dependency and to preserve outbound connectivity for private EC2 instances.

An S3 Gateway Endpoint allows EC2 instances to reach Amazon S3 without routing S3 traffic through the NAT Gateways. This reduces NAT data processing and aligns with the Cost Optimization pillar.

**2. Backend compute and storage**

Two `t3.small` EC2 instances keep the Backend available across two Availability Zones. The Auto Scaling Group maintains at least two instances while the environment is active, allowing the ALB to retain a healthy target if one instance fails. Each instance uses a separate gp3 EBS root volume.

During the workshop phase, EC2 runs only for learning, acceptance testing, and demonstrations. The Auto Scaling Group is scaled to zero after each session so that compute hours do not continue accumulating.

**3. Application Load Balancer**

The ALB is the HTTPS entry point for the Backend and forwards requests to the Target Group on port 5000. Charges include ALB runtime and consumed LCUs. Since acceptance traffic is low, the LCU component is much smaller than the hourly charge.

To retain the USD 50–60 target, the ALB must be deleted after workshop sessions and recreated from documented configuration when needed. Keeping the ALB for 730 hours per month would increase the estimate.

**4. Frontend, media, and content delivery**

Static frontend files are stored in S3 and delivered through CloudFront. Media files are stored in a separate bucket and accessed through presigned URLs. This prevents large video payloads from passing through EC2, reduces Backend load, and makes S3 cost proportional to storage and request volume.

CloudFront currently uses the Free plan, and workshop traffic remains below its allowance. The table therefore contains no additional CloudFront charge. The estimate must be updated when traffic or student usage grows.

**5. Logging and management services**

CloudWatch centralizes logs from the EC2 instances in the Auto Scaling Group and evaluates alarms. IAM, ACM, SSM Parameter Store, and the S3 Gateway Endpoint have no base resource charge under the current configuration, although customer-managed KMS keys, advanced parameters, or high API volume may incur additional charges.


#### Cost control and optimization actions

1. Maintain an AWS Budget with alerts at 50%, 80%, and 100%.
2. Review Cost Explorer by service, especially `EC2-Other`, NAT Gateway, and Elastic Load Balancing.
3. Scale the Auto Scaling Group to zero after demonstrations.
4. Delete unused ALBs, Target Groups, NAT Gateways, and release Elastic IPs when the environment is paused.
5. Route S3 traffic through the S3 Gateway Endpoint.
6. Configure an ECR lifecycle policy and retain only images required for rollback.
7. Configure CloudWatch Logs retention instead of storing logs indefinitely.
8. Track S3 Media growth and introduce lifecycle rules as older video content accumulates.
9. Before continuous production, evaluate EC2 Savings Plans and more cost-efficient egress patterns.

#### Conclusion

With controlled runtime and disciplined resource cleanup, LearnSphere can retain its Multi-AZ architecture for workshop use at approximately **USD 56.09 per month**. The estimate explicitly accounts for NAT Gateway and ALB charges, does not depend on an assumed promotional credit, and clearly separates workshop operation from continuous production.

