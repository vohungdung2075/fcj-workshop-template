---
title: "Launch Template and User Data bootstrap"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.5.3. </b> "
---

#### Objective

The Launch Template ensures that every Backend instance uses the same configuration. The ASG does not clone a running server; it launches a clean instance from the template, while User Data reconstructs runtime from Parameter Store and ECR.

#### 1. Configure the Launch Template

`LearnSphere-Backend-LT`:

| Attribute | Value |
| --- | --- |
| Launch Template ID | `lt-0f60c47d6fe12d8fc` |
| Default version | `2` |
| AMI | Amazon Linux 2023 |
| Deployment-time AMI ID | `ami-01b70d44184a858e8` |
| Instance type | `t3.small` |
| Instance profile | `LearnSphereEc2Role2` |
| Security Group | `LearnSphere-Backend-SG` |
| Public IP | Disabled |
| Key pair | Not required |

Do not select a fixed subnet in the Launch Template. The ASG selects from the two private subnets and distributes instances across Availability Zones.

#### 2. User Data principles

User Data must:

* Run without interaction.
* Stop when a critical command fails.
* Contain no hard-coded secret.
* Work on a completely new instance.
* Finish only after the readiness endpoint returns HTTP 200.

```bash
#!/bin/bash
set -euxo pipefail

REGION=ap-southeast-1
REGISTRY=440893644584.dkr.ecr.ap-southeast-1.amazonaws.com
REPOSITORY=learnsphere-be-2

dnf install -y docker
systemctl enable --now docker

aws ssm get-parameter \
  --region "$REGION" \
  --name /learnsphere/prod/backend-env \
  --with-decryption \
  --query Parameter.Value \
  --output text > /home/ec2-user/.env

chmod 600 /home/ec2-user/.env

IMAGE_TAG=$(aws ssm get-parameter \
  --region "$REGION" \
  --name /learnsphere/prod/backend-image-tag \
  --query Parameter.Value \
  --output text)

aws ecr get-login-password --region "$REGION" |
  docker login --username AWS --password-stdin "$REGISTRY"

docker pull "$REGISTRY/$REPOSITORY:$IMAGE_TAG"

docker run -d \
  --name learnsphere-be \
  --restart unless-stopped \
  --env-file /home/ec2-user/.env \
  -p 5000:5000 \
  --log-driver awslogs \
  --log-opt awslogs-region="$REGION" \
  --log-opt awslogs-group=/learnsphere/backend2 \
  --log-opt awslogs-stream="backend-$(hostname)" \
  "$REGISTRY/$REPOSITORY:$IMAGE_TAG"

curl --fail --retry 30 --retry-delay 10 \
  http://127.0.0.1:5000/health/ready
```

Enter this script as plain text in the AWS Console. Do not select “User data has already been base64 encoded” unless the script has actually been encoded.

#### 3. Bootstrap sequence

1. Amazon Linux starts cloud-init.
2. Docker is installed and enabled.
3. The EC2 role retrieves the environment SecureString.
4. The `.env` file receives mode `600`.
5. The EC2 role reads the production image tag.
6. Docker authenticates to ECR with a temporary token.
7. The exact commit-SHA image is pulled.
8. The container starts with restart policy `unless-stopped`.
9. Docker sends stdout/stderr to `/learnsphere/backend2`.
10. The script retries readiness during startup.

#### 4. Why version 2 is used

Version 1 existed before the User Data fully installed Docker. EC2 health was successful, but no container listened on port 5000 and the Target Group remained unhealthy. Version 2 added complete bootstrap logic and became the default before the ASG recreated both instances.

Creating a new version instead of silently replacing configuration preserves history and provides a rollback path.

#### 5. Validate a new instance

Through Systems Manager Run Command:

```bash
sudo systemctl status docker --no-pager
sudo docker ps --filter name=learnsphere-be
curl -fsS http://127.0.0.1:5000/health/ready
```

Docker must be active, the container must be running, and readiness must report `database=connected`.

#### Deployment evidence

![LearnSphere Backend Launch Template version 2](/images/learnsphere-launch-template-v2.png)

*Figure 5.22. `LearnSphere-Backend-LT` uses version 2 as the default, Amazon Linux AMI `ami-01b70d44184a858e8`, instance type `t3.small`, and Backend Security Group `sg-0cae310ddce032bbf`. The complete User Data is intentionally excluded because operational configuration must not expose sensitive values.*
