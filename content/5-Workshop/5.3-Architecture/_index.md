---
title: "Architecture and system flows"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Design objectives

The LearnSphere production architecture follows a layered model that separates presentation, API, data, and integration responsibilities. Its primary objectives are:

* Deliver the Frontend over HTTPS with low latency and SPA navigation support.
* Prevent direct public access to Backend EC2 instances and S3 buckets.
* Maintain at least two Backend instances across two Availability Zones.
* Keep state outside EC2 so that instances can be replaced safely.
* Upload videos, documents, and thumbnails directly to S3 instead of proxying large files through the Backend.
* Automate testing, packaging, and deployment through GitHub Actions.
* Centralize logs, metrics, health checks, and operational alerts.

The architecture runs in the Singapore Region (`ap-southeast-1`). The CloudFront certificate resides in `us-east-1` as required by the service, while the Application Load Balancer certificate resides in `ap-southeast-1`.

#### Architecture

![LearnSphere Multi-AZ production architecture](/images/LEARNSHPHERE.png)

*Figure 5.6. LearnSphere As-Built architecture after the Multi-AZ high-availability Backend upgrade.*
> **Future Architecture Evolution (AWS Native Roadmap):**  
> In future iterations, our team plans to migrate **MongoDB Atlas to Amazon DynamoDB** (utilizing the `Dynamoose` ODM library connected internally via VPC Gateway Endpoints) and migrate **Groq API to Amazon Bedrock** (direct Claude 3.5 access with built-in Bedrock Guardrails & Knowledge Bases for RAG). Additionally, integrating **AWS WAF** at the edge and employing VPC Interface Endpoints will optimize security and infrastructure costs.


The diagram uses two types of connections:

* **Solid lines** represent runtime traffic and data exchanged by users, the Backend, and monitoring services.
* **Dashed lines** represent administrative, permission, configuration, and deployment flows initiated by GitHub Actions.

#### Architecture layers

| Layer | Components | Responsibility |
| --- | --- | --- |
| Client and DNS | Browser and TenTen DNS | Resolve `www.learnspherev2.id.vn` and send HTTPS requests |
| Edge | Amazon CloudFront and AWS Certificate Manager | Deliver the Frontend, terminate TLS, and route `/api/*` |
| Frontend | Amazon S3 Frontend | Store the React/Vite static build; the private bucket is readable only by CloudFront OAC |
| Network | VPC, public/private subnets, Internet Gateway, NAT Gateways, and S3 Gateway Endpoint | Isolate the Backend and provide controlled ingress and egress |
| API entry | Application Load Balancer | Receive HTTPS from CloudFront and distribute requests to the Target Group |
| Compute | Auto Scaling Group, Launch Template, EC2, and Docker | Run the Express.js Backend on port 5000 and replace failed instances |
| Media | Amazon S3 Media | Store videos, documents, thumbnails, and avatars through presigned URLs |
| Configuration | AWS Systems Manager Parameter Store | Store the production environment as a SecureString and the released image tag |
| Container registry | Amazon ECR | Store immutable Backend Docker images tagged by commit SHA |
| Data | MongoDB Atlas | Store users, courses, lessons, quizzes, enrollments, progress, and AI records |
| AI | Groq API | Provide chat, document summarization, and quiz generation |
| Observability | Amazon CloudWatch and Amazon SNS | Collect logs and metrics, evaluate alarms, and send email notifications |
| Delivery | GitHub Actions and IAM OIDC | Test, build, and deploy without long-lived access keys |

#### Multi-AZ network design

LearnSphere uses the `10.20.0.0/16` VPC with DNS resolution and DNS hostnames enabled. Resources are distributed across two Availability Zones:

| Availability Zone | Public subnet | Private subnet | Main resources |
| --- | --- | --- | --- |
| `ap-southeast-1a` | Public subnet 1a | Private subnet 1a | ALB node, NAT Gateway 1a, and Backend EC2 1a |
| `ap-southeast-1b` | Public subnet 1b | Private subnet 1b | ALB node, NAT Gateway 1b, and Backend EC2 1b |

Both public subnets have a `0.0.0.0/0` route to the Internet Gateway. Each private subnet has a default route to the NAT Gateway in the same Availability Zone. This arrangement prevents the Backend in one AZ from depending on the other AZ's NAT Gateway.

The Application Load Balancer is attached to both public subnets so that it can accept HTTPS connections. Backend EC2 instances run only in private subnets, have no public IPv4 addresses, and expose no inbound SSH. They can still pull images, call external APIs, and send email through their NAT Gateways.

An S3 Gateway Endpoint is associated with both private route tables. EC2-to-S3 traffic therefore remains on the AWS network rather than crossing a NAT Gateway, which reduces Internet exposure and NAT data-processing costs.

![LearnSphere VPC Resource Map](/images/learnsphere-vpc-resource-map.png)

*Figure 5.7. The VPC Resource Map shows four subnets across two Availability Zones, their route tables, the Internet Gateway, two NAT Gateways, and the S3 Gateway Endpoint.*

#### Frontend and API access flow

The application and API request path works as follows:

1. A user visits `https://www.learnspherev2.id.vn`.
2. TenTen DNS resolves the name to the CloudFront distribution.
3. CloudFront uses an ACM certificate to establish HTTPS.
4. For UI requests, the default behavior retrieves `index.html` and static assets from the private Frontend S3 bucket through Origin Access Control.
5. A CloudFront Function rewrites SPA paths such as `/profile`, `/courses`, and `/system-monitoring` to `index.html`.
6. When the Frontend calls `/api/*`, CloudFront forwards the request over HTTPS to `origin.learnspherev2.id.vn`.
7. The origin domain resolves to `LearnSphere-Prod-ALB`.
8. The ALB terminates TLS, evaluates its listener rule, and forwards HTTP port 5000 traffic to `LearnSphere-Backend-TG`.
9. The Target Group sends requests only to healthy EC2 targets in the two private subnets.
10. The Backend validates the JWT/cookie, executes business logic, and accesses MongoDB Atlas or another required service.
11. The response returns through EC2 → ALB → CloudFront → browser.

CloudFront is the unified entry point for both the UI and the API. The Frontend can call the same-origin `/api/*` path, reducing CORS configuration and preventing users from needing to know the ALB DNS name.

#### Learning-material upload and download flow

Videos and documents can be large, so LearnSphere does not proxy their contents through Express.js. It uses the following presigned URL flow:

1. An authenticated user asks the Backend to create an upload session.
2. The Backend validates the role, course ownership, file type, size, and S3 object key.
3. The Backend uses the EC2 instance role to create a presigned URL or multipart presigned URLs.
4. The browser uploads the file directly to S3 Media.
5. After a successful upload, the Frontend calls the session-completion API.
6. The Backend verifies the S3 object and stores its key and metadata in MongoDB.
7. When a lesson is opened, the Backend returns a time-limited presigned download URL to an authorized user.

This flow reduces CPU, memory, and network pressure on EC2. S3 Media keeps Block Public Access enabled, while signed URLs provide time-bound object access. Cleanup jobs handle expired multipart sessions and orphaned objects to reduce unnecessary storage costs.

#### Data and AI processing flow

MongoDB Atlas is the shared data source for all Backend instances. EC2 does not keep durable business data on local disks, allowing the ASG to terminate or replace instances without losing courses, lessons, quizzes, or progress.

When an instructor requests document summarization or AI-generated quiz questions:

1. The Backend loads document metadata from MongoDB.
2. It reads the corresponding object from S3 Media.
3. Text is extracted with `pdf-parse`, `mammoth`, or `tesseract.js` OCR for scanned PDFs.
4. The text is normalized and limited before being sent to the Groq API.
5. The AI output is structurally validated and then returned to the Frontend or stored in MongoDB, depending on the feature.
6. The `model_id`, input-token count, and output-token count are recorded for usage tracking.

Student chat also passes through the Backend for authentication, rate limiting, lesson-context selection, and history persistence. The Groq API key is never exposed to the browser.

The Backend in AZ 1a reaches the Internet through NAT Gateway 1a, while the instance in AZ 1b uses NAT Gateway 1b. Both NAT Elastic IPs are allowlisted in MongoDB Atlas Network Access. Groq and email services are reached over TLS from these stable egress addresses.

#### EC2 configuration and bootstrap flow

The Launch Template defines the AMI, instance type, Backend Security Group, EC2 instance profile, and User Data. When the ASG creates a new instance, bootstrap performs the following steps:

1. Install and start Docker.
2. Read the production environment from `/learnsphere/prod/backend-env` in Parameter Store.
3. Read the currently released image tag from `/learnsphere/prod/backend-image-tag`.
4. Authenticate to ECR using temporary EC2 instance-role credentials.
5. Pull the exact Docker image released by the pipeline.
6. Start the container with the CloudWatch Logs driver.
7. Call `/health/ready` and complete only after MongoDB is connected.

The environment is stored as an SSM SecureString, while the image tag is maintained as a separate String parameter. GitHub Actions does not need to read production secret values; each EC2 instance retrieves its configuration through a least-privilege instance role.

#### CI/CD flow

For each deployment, GitHub Actions obtains temporary credentials through OIDC and an IAM role instead of storing long-lived `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` values.

```text
GitHub Actions
  → assume IAM role through OIDC
  → run Backend tests
  → build Docker image
  → push image tagged with commit SHA to ECR
  → update the candidate image tag in Parameter Store
  → start an ASG Instance Refresh
  → wait for healthy EC2 instances and Target Group
  → verify /health/ready through the ALB
  → build the React/Vite Frontend
  → synchronize dist to the Frontend S3 bucket
  → create a CloudFront invalidation
```

Instance Refresh follows a launch-before-terminate strategy. A new instance must pass health checks before the old instance is removed. If the candidate does not become healthy, the workflow stops, restores the previous image tag, and preserves the serving capacity.

#### High availability and self-healing

`LearnSphere-Backend-ASG` uses the following settings:

| Property | Value |
| --- | --- |
| Minimum capacity | 2 |
| Desired capacity | 2 |
| Maximum capacity | 4 |
| Health checks | EC2 and ELB |
| Distribution | Two private subnets across two AZs |
| Maintenance | Launch before terminating |

![Healthy LearnSphere Backend Target Group](/images/learnsphere-target-group-healthy.png)

*Figure 5.8. The Application Load Balancer Target Group reports two healthy Backend targets on port 5000 and no unhealthy targets.*

Under normal conditions, each AZ runs one Backend EC2 instance. If a container, instance, or AZ target fails:

1. The readiness check fails and the ALB stops routing traffic to that target.
2. Requests continue to use the remaining healthy target.
3. The ASG receives the ELB health result and launches a replacement.
4. The new instance bootstraps from the same Launch Template, Parameter Store values, and ECR image.
5. After the target becomes healthy, the ASG returns to its desired capacity.

The ALB and ASG remove the single point of failure at the Backend tier. S3 and CloudFront are highly available managed services, while the two NAT Gateways provide per-AZ egress.

The `max=4` value is a capacity ceiling and does not automatically enable scale-out. The current architecture maintains and self-heals two instances. Automatic expansion and contraction require a target-tracking or step-scaling policy based on CPU, request count, or response time.

#### Security boundaries

| Boundary | Implemented configuration |
| --- | --- |
| HTTPS | ACM for CloudFront in `us-east-1` and ACM for the ALB in `ap-southeast-1` |
| ALB Security Group | Accept HTTPS on port 443 and forward to the Backend Target Group |
| Backend Security Group | Accept TCP port 5000 only from the ALB Security Group |
| EC2 | Private subnets, no public IP, and no inbound SSH |
| S3 Frontend | Block Public Access with CloudFront OAC as the only reader |
| S3 Media | Block Public Access with presigned uploads and downloads |
| IAM | Separate GitHub deploy role, EC2 instance role, and ASG service-linked role |
| Secrets | Production environment in SSM SecureString; `.env` is not committed |
| Database | MongoDB Atlas allowlists only the two NAT Elastic IPs |
| Application | HTTPS-origin CORS, JWT secret, security headers, and rate limiting |

#### Monitoring and alerts

Backend containers send stdout and stderr to CloudWatch Logs. The liveness endpoint reports whether the Node.js process is running, while the readiness endpoint returns `ready` only when MongoDB is available. The ALB uses readiness to determine whether a target can receive traffic.

CloudWatch provides EC2, ASG, and ALB metrics. Alarms can monitor unhealthy targets, InService instance count, HTTP 5xx responses, response time, and CPU. When an alarm changes state, Amazon SNS sends an email notification through the `LearnSphere-Alerts` topic.

#### AWS Well-Architected assessment

| Pillar | LearnSphere implementation |
| --- | --- |
| Operational Excellence | Automated CI/CD, immutable images, health checks, centralized logs, and rollback workflow |
| Security | End-to-end HTTPS, private subnets, OAC, presigned URLs, IAM roles, and SSM SecureString |
| Reliability | Two AZs, ALB, ASG self-healing, two NAT Gateways, and state externalized from EC2 |
| Performance Efficiency | CloudFront caching, direct S3 uploads, and an S3 Gateway Endpoint |
| Cost Optimization | AWS Budget, S3 lifecycle/cleanup, S3 endpoint to avoid NAT processing, and bounded capacity |
| Sustainability | Static hosting, CDN caching, replaceable containers, and reduced file transfer through compute |

#### Current scope and limitations

High availability is currently implemented within one AWS Region across two Availability Zones; this is not yet a multi-Region disaster-recovery architecture. MongoDB Atlas and Groq remain dependencies outside the AWS account, so end-to-end availability still depends on the Atlas cluster configuration and the Groq service.

In the next development phase, LearnSphere plans to migrate its data layer from MongoDB Atlas to Amazon DynamoDB. This will bring business data into the AWS ecosystem, provide serverless scaling, and remove database-server administration. The migration must be implemented incrementally by analyzing access patterns, designing partition and sort keys, replacing the Mongoose data-access layer, building migration tooling, validating consistency, and switching traffic only after the migrated data has been verified.

The AI layer is also planned to migrate from Groq to Amazon Bedrock. This will reduce reliance on an external provider, standardize authorization through IAM, and bring AI inference into the same AWS monitoring, security, and cost-governance environment. The transition will reuse the existing AI-provider abstraction, configure a suitable Bedrock model, grant invoke permissions according to least privilege, verify service quotas, and compare response quality, latency, token usage, and cost against Groq. Groq will be retired only after the learning assistant, document summarization, and quiz-generation workflows pass regression testing on Bedrock.

The infrastructure roadmap also includes Amazon Route 53 for centralized DNS and health checks; AWS WAF at CloudFront to filter malicious requests, rate-based attacks, and common exploits; ASG target tracking for load-based scaling; AWS Secrets Manager for secret management and rotation; and multi-Region backup, restore, and failover planning. These improvements provide a path from the current Multi-AZ architecture toward a more secure, scalable, and resilient cloud-native platform.
