---
title: "Deploying the highly available Backend"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Objectives

Section 5.4 established IAM, storage, and the Multi-AZ network. Section 5.5 deploys the Node.js/Express Backend on that foundation through an Application Load Balancer and an Auto Scaling Group spanning two Availability Zones.

The completed deployment must provide:

* Immutable Backend Docker images from Amazon ECR.
* Production secrets outside the Docker image and Launch Template.
* Two Backend EC2 instances in separate private subnets.
* ALB routing only to targets that pass readiness checks.
* An ASG that maintains at least two instances and replaces failed instances.
* Fully automatic instance bootstrap without SSH or manual configuration.
* Launch-before-terminate replacement during application releases.

#### Production components

| Component | Resource |
| --- | --- |
| Runtime configuration | `/learnsphere/prod/backend-env` |
| Released image tag | `/learnsphere/prod/backend-image-tag` |
| Application Load Balancer | `LearnSphere-Prod-ALB` |
| Target Group | `LearnSphere-Backend-TG` |
| Launch Template | `LearnSphere-Backend-LT`, default version `2` |
| Auto Scaling Group | `LearnSphere-Backend-ASG` |
| Backend instances | Two private `t3.small` EC2 instances |
| Health endpoint | `GET /health/ready` |
| Backend origin | `https://origin.learnspherev2.id.vn` |

#### Implementation pages

1. [Runtime configuration in Parameter Store](5.5.1-runtime-configuration/)
2. [Target Group and Application Load Balancer](5.5.2-alb-target-group/)
3. [Launch Template and User Data bootstrap](5.5.3-launch-template/)
4. [Multi-AZ Auto Scaling Group](5.5.4-auto-scaling/)
5. [High Availability and self-healing validation](5.5.5-ha-validation/)

#### Backend instance bootstrap flow

```text
Auto Scaling Group
→ select a private subnet
→ launch EC2 from the Launch Template
→ read the environment and image tag from Parameter Store
→ authenticate to ECR with the instance role
→ pull the commit-SHA Docker image
→ run the container on port 5000
→ send logs to CloudWatch
→ return HTTP 200 from /health/ready
→ Target Group marks the target Healthy
→ ALB starts routing requests
```

#### Completion criteria

The Backend HA layer is complete when both instances are `InService` and `Healthy`, the Target Group has two healthy targets in separate Availability Zones, the ALB returns HTTP 200 from the readiness endpoint, and one instance can be replaced without removing all service capacity.

ASG `maximum=4` is a capacity boundary rather than an automatic scale-out policy. The current architecture provides HA and self-healing at desired capacity 2. Load-driven elasticity requires a future Target Tracking or Step Scaling policy.
