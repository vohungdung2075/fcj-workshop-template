---
title: "Runtime configuration in Parameter Store"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

#### Objective

Backend instances in the ASG are replaceable and short-lived. Runtime configuration must therefore not be stored manually on individual EC2 instances or baked into Docker images. LearnSphere uses AWS Systems Manager Parameter Store as the centralized configuration source.

#### 1. Separate environment and image version

| Parameter | Type | Reader/writer |
| --- | --- | --- |
| `/learnsphere/prod/backend-env` | `SecureString` | Read by `LearnSphereEc2Role2`; updated by an authorized operator |
| `/learnsphere/prod/backend-image-tag` | `String` | Read by EC2; updated and rolled back by GitHub Actions |

`backend-env` contains secrets and runtime settings. `backend-image-tag` contains only the 40-character Git commit SHA of the released Docker image.

#### 2. Prepare the production environment

Important variable groups include:

```dotenv
PORT=5000
NODE_ENV=production
TRUST_PROXY=1
MONGODB_URI=<secret>
JWT_SECRET=<secret-at-least-64-characters>
FRONTEND_URL=https://www.learnspherev2.id.vn
AWS_REGION=ap-southeast-1
AWS_S3_BUCKET=learnsphere-media-2
AI_PROVIDER=groq
GROQ_API_KEY=<secret>
GROQ_MODEL=<model>
```

`AUTH_COOKIE_DOMAIN` may remain empty for host-only cookies when Frontend and API share `www.learnspherev2.id.vn`. Bedrock variables do not participate in runtime when `AI_PROVIDER=groq`.

#### 3. Create the SecureString

From a host with AWS CLI and appropriate permissions:

```bash
aws ssm put-parameter \
  --region ap-southeast-1 \
  --name "/learnsphere/prod/backend-env" \
  --description "LearnSphere production backend environment" \
  --type SecureString \
  --tier Standard \
  --key-id alias/aws/ssm \
  --value file:///home/ec2-user/.env \
  --overwrite
```

AWS managed key `alias/aws/ssm` encrypts the value. Only the EC2 instance role and authorized operators can call `GetParameter` with `WithDecryption=true`.

#### 4. Create the image-tag parameter

Initialize it with a commit SHA that already exists in ECR:

```bash
aws ssm put-parameter \
  --region ap-southeast-1 \
  --name "/learnsphere/prod/backend-image-tag" \
  --type String \
  --value "<40-character-git-sha>" \
  --overwrite
```

The pipeline reads the previous tag before release. If Instance Refresh or the public health check fails, the workflow restores the previous tag and starts a rollback refresh.

#### 5. Validate without exposing secrets

Inspect only metadata or length:

```bash
aws ssm get-parameter \
  --region ap-southeast-1 \
  --name "/learnsphere/prod/backend-env" \
  --with-decryption \
  --query "length(Parameter.Value)" \
  --output text
```

Inspect the image tag:

```bash
aws ssm get-parameter \
  --region ap-southeast-1 \
  --name "/learnsphere/prod/backend-image-tag" \
  --query "Parameter.{Name:Name,Type:Type,Version:Version,Value:Value}" \
  --output table
```

Never reveal the MongoDB URI, JWT secret, email password, or Groq API key in screenshots, terminal history, workflow logs, or the report.

#### 6. MongoDB Atlas Network Access

Atlas allowlists both production NAT Elastic IPs:

```text
54.179.11.158/32
52.221.42.74/32
```

Any replacement EC2 instance can therefore reach Atlas through its local NAT Gateway without adding per-instance addresses.

#### Deployment evidence

![LearnSphere production Backend parameters in Systems Manager Parameter Store](/images/learnsphere-ssm-backend-parameters.png)

*Figure 5.19. Parameter Store contains `/learnsphere/prod/backend-env` as a `SecureString` and `/learnsphere/prod/backend-image-tag` as a `String`. The screenshot proves that runtime configuration and the deployable image tag are managed centrally without exposing decrypted secret values.*
