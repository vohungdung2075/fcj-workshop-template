---
title: "Building the core AWS infrastructure"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Objectives

This section builds the infrastructure foundation required before deploying the highly available Backend in Section 5.5. The resources follow separation-of-duties, network-segmentation, and private-by-default principles so that application servers and stored data are not directly exposed.

After Section 5.4, the production environment provides:

* Temporary AWS credentials for GitHub Actions through OIDC, without long-lived access keys.
* A dedicated EC2 instance profile for configuration retrieval, image pulls, and media access.
* Two private S3 buckets that separate Frontend artifacts from learning media.
* An ECR repository that stores Backend Docker images by Git commit SHA.
* A VPC spanning two Availability Zones with two public and two private subnets.
* One NAT Gateway per Availability Zone for independent outbound connectivity.
* An S3 Gateway Endpoint that keeps private-subnet S3 traffic away from NAT.
* Security Groups that allow the ALB to reach the Backend only on port 5000.

#### Resource scope

| Group | Production resource |
| --- | --- |
| AWS Account | `440893644584` |
| Region | Singapore (`ap-southeast-1`) |
| VPC | `LearnSphere-Prod-vpc` — `10.20.0.0/16` |
| Public subnets | Two subnets in `ap-southeast-1a` and `ap-southeast-1b` |
| Private subnets | Two subnets in `ap-southeast-1a` and `ap-southeast-1b` |
| Container registry | ECR repository `learnsphere-be-2` |
| Static Frontend | S3 bucket `learnsphere-fe-2` |
| Media | S3 bucket `learnsphere-media-2` |
| Deployment identity | `LearnSphereGitHubDeployRole2` |
| Runtime identity | `LearnSphereEc2Role2` |
| Network security | `LearnSphere-ALB-SG`, `LearnSphere-Backend-SG` |

#### Implementation pages

1. [IAM, GitHub OIDC, and runtime permissions](5.4.1-iam/)
2. [Amazon S3 and Amazon ECR](5.4.2-storage-ecr/)
3. [Multi-AZ VPC network](5.4.3-network/)
4. [Security Groups and infrastructure validation](5.4.4-security/)

This sequence ensures that later resources can reference existing roles, buckets, repositories, subnets, and Security Groups. The ALB, Target Group, Launch Template, and Auto Scaling Group are implemented in Section 5.5.

#### Completion criteria

The core infrastructure is ready when:

* The GitHub OIDC trust policy is restricted to the intended repository and `main` branch.
* Block Public Access remains enabled on both S3 buckets.
* ECR contains a Docker image tagged with a commit SHA.
* Each private subnet routes outbound traffic through the NAT Gateway in the same AZ.
* Both private route tables use the S3 Gateway Endpoint.
* The Backend Security Group does not expose port 5000 to `0.0.0.0/0`.
* Both NAT Elastic IPs are allowlisted in MongoDB Atlas.

#### Result

The resulting foundation establishes clear boundaries between the edge, public ingress, private compute, and data services. Auto Scaling can create or replace Backend EC2 instances without assigning public IP addresses, distributing SSH keys, or configuring each server manually.
