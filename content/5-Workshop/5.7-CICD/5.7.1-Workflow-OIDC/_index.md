---
title: "Workflow, OIDC, and deployment permissions"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.7.1. </b> "
---

#### 1. Trigger and GitHub permissions

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

concurrency:
  group: learnsphere-production
  cancel-in-progress: false
```

`contents: read` is sufficient to check out the source. `id-token: write` lets the runner request an OIDC token; it does not grant source write access.

#### 2. Assume the role through OIDC

```yaml
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: ${{ secrets.AWS_GITHUB_ROLE_ARN }}
    aws-region: ap-southeast-1
```

The `LearnSphereGitHubDeployRole2` trust policy restricts:

* Provider: `token.actions.githubusercontent.com`.
* Audience: `sts.amazonaws.com`.
* Subject: the LearnSphere repository and `main` branch.

AWS STS issues temporary credentials for each job. They expire when the job ends, reducing the risk of stored access keys.

#### 3. Deployment permission scope

The role needs only:

* Image operations for `learnsphere-be-2`.
* Read/write access to `/learnsphere/prod/backend-image-tag`.
* ASG inspection and Instance Refresh.
* Object synchronization to `learnsphere-fe-2`.
* Invalidation for the designated distribution.

EC2 uses a separate `LearnSphereEc2Role2` runtime role to read configuration, pull images, access S3 Media, and write CloudWatch Logs. Runtime and deployment roles are not shared.

#### 4. GitHub variables and secrets

| Name | Type | Purpose |
| --- | --- | --- |
| `AWS_GITHUB_ROLE_ARN` | Secret | Role assumed through GitHub OIDC |
| `S3_FE_BUCKET` | Secret/Variable | Frontend bucket |
| `CLOUDFRONT_FE_DISTRIBUTION_ID` | Secret/Variable | Distribution to invalidate |
| `VITE_API_BASE_URL` | Secret/Variable | `/api` in production |

Screenshots should show names only; neither GitHub nor this report should reveal sensitive values.

#### Deployment evidence

![GitHub Actions workflow that deploys LearnSphere to AWS](/images/learnsphere-github-actions-workflow.png)

*Figure 5.39. The production workflow runs on pushes to `main` or manual dispatch, serializes deployments, grants OIDC permission, and configures backend rollout through the Auto Scaling Group.*

![Repository Secrets used by the production workflow](/images/learnsphere-github-repository-secrets.png)

*Figure 5.40. GitHub displays only the names of the four repository secrets required for OIDC, S3, CloudFront, and API configuration; their values are not exposed in the interface or report.*
