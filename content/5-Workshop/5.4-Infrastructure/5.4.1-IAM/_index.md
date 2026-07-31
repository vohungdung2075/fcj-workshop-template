---
title: "IAM, GitHub OIDC, and runtime permissions"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

#### Objective

LearnSphere separates the deployment identity from the runtime identity:

| Role | Principal | Purpose |
| --- | --- | --- |
| `LearnSphereGitHubDeployRole2` | GitHub Actions through OIDC | Build and release the application |
| `LearnSphereEc2Role2` | Amazon EC2 | Bootstrap and operate the Backend |

This separation prevents the pipeline from reading all runtime secrets and prevents EC2 instances from changing CloudFront or deploying the Frontend.

#### 1. Create the GitHub OIDC provider

In the IAM Console, open **Identity providers → Add provider** and enter:

| Field | Value |
| --- | --- |
| Provider type | OpenID Connect |
| Provider URL | `https://token.actions.githubusercontent.com` |
| Audience | `sts.amazonaws.com` |

OIDC allows GitHub Actions to exchange a workflow token for short-lived AWS credentials. The repository does not store `AWS_ACCESS_KEY_ID` or `AWS_SECRET_ACCESS_KEY`.

#### 2. Create the deployment role

Create `LearnSphereGitHubDeployRole2` with a trust policy restricted to the intended repository, audience, and production branch:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::440893644584:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:HoiaeKHMT/LearnSphere:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

The `sub` condition prevents workflows from forks, other repositories, or other branches from assuming this role.

#### 3. Grant least-privilege deployment permissions

The deployment role is organized by task:

| Permission group | Required actions |
| --- | --- |
| ECR | Obtain an authorization token, inspect layers, upload layers, and push images |
| S3 Frontend | List the bucket and upload, read, or remove stale objects during `s3 sync --delete` |
| CloudFront | Create and inspect invalidations |
| Parameter Store | Read and update `/learnsphere/prod/backend-image-tag` |
| Auto Scaling | Describe the ASG and create or monitor Instance Refreshes |
| EC2/ELB read-only | Inspect instances and Target Group health during validation |

Where supported, resource ARNs are restricted to `learnsphere-be-2`, `learnsphere-fe-2`, the production CloudFront distribution, the Parameter Store path, and `LearnSphere-Backend-ASG`.

#### 4. Create the EC2 instance role

Create `LearnSphereEc2Role2` and attach it through the instance profile used by the Launch Template. The runtime role needs to:

* Read and decrypt `/learnsphere/prod/backend-env`.
* Read `/learnsphere/prod/backend-image-tag`.
* Obtain an ECR authorization token and pull from `learnsphere-be-2`.
* Read, write, and remove objects in `learnsphere-media-2` as required by application workflows.
* Send container logs to CloudWatch Logs.
* Connect to Systems Manager so operators do not need SSH.

The runtime role does not need to deploy Frontend files, invalidate CloudFront, or modify the Auto Scaling Group.

#### 5. Store configuration in Parameter Store

Two parameters are maintained independently:

| Parameter | Type | Content |
| --- | --- | --- |
| `/learnsphere/prod/backend-env` | `SecureString` | Complete production Backend environment |
| `/learnsphere/prod/backend-image-tag` | `String` | Git commit SHA currently released |

Separating the image tag lets the pipeline change Backend versions without reading or rewriting the MongoDB URI, JWT secret, email password, or Groq API key.

```bash
aws ssm put-parameter \
  --region ap-southeast-1 \
  --name "/learnsphere/prod/backend-env" \
  --description "LearnSphere production backend environment" \
  --type SecureString \
  --key-id alias/aws/ssm \
  --value file://.env \
  --overwrite
```

#### 6. Validate the identities

The GitHub workflow must include:

```yaml
permissions:
  id-token: write
  contents: read
```

After `aws-actions/configure-aws-credentials`, run:

```bash
aws sts get-caller-identity
```

The result should identify `LearnSphereGitHubDeployRole2`. Running the same command on a Backend EC2 instance should identify `LearnSphereEc2Role2`.

#### IAM configuration results

![LearnSphere GitHub OIDC provider](/images/learnsphere-github-oidc-role.png)

*Figure 5.9. The `token.actions.githubusercontent.com` GitHub OIDC provider uses OpenID Connect with `sts.amazonaws.com` as its audience. The repository and branch restrictions are documented in the trust-policy code above.*

![LearnSphere IAM roles](/images/learnsphere-iam-roles.png)

*Figure 5.10. IAM roles are separated by function: `LearnSphereGitHubDeployRole2` supports CI/CD, while `LearnSphereEc2Role2` is assumed by the Backend EC2 instances.*

These results confirm that LearnSphere does not share one identity between deployment and runtime. Each role receives only the permissions required for its responsibility, reducing the blast radius of a compromised component.
