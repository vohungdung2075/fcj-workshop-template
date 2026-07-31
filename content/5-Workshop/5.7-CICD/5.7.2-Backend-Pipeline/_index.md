---
title: "Backend release pipeline"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.7.2. </b> "
---

#### 1. Validate source

The backend job uses Node.js 24:

```bash
cd LearnSphere_BE
npm ci
npm test
```

Tests cover the quiz parser, health endpoints, JSON 404 behavior, CORS, and production environment validation. The pipeline stops immediately if a test fails.

#### 2. Build an immutable image

```text
ECR URI: 440893644584.dkr.ecr.ap-southeast-1.amazonaws.com/learnsphere-be-2
Image tag: $GITHUB_SHA
```

The workflow checks whether the SHA image already exists. It builds and pushes only when necessary; otherwise, it reuses the exact artifact. `latest` is not used, so every release and rollback maps to a source commit.

#### 3. Pre-deployment guards

Before rollout, the script verifies:

* The SHA is exactly 40 hexadecimal characters.
* The image exists in ECR.
* No other Instance Refresh is active.
* Desired capacity is at least two.
* Healthy instance count equals desired capacity.

An unsafe baseline stops the workflow before instance replacement.

#### 4. Update the release and refresh instances

The pipeline saves the old tag and writes the new SHA to:

```text
/learnsphere/prod/backend-image-tag
```

It then starts Instance Refresh:

```text
MinHealthyPercentage = 100
MaxHealthyPercentage = 200
InstanceWarmup        = 300
SkipMatching          = false
```

With launch-before-terminate, the ASG creates a new instance, waits for a healthy target, and only then removes the old one. The workflow waits up to 60 minutes and does not treat “refresh started” as a successful deployment.

#### Deployment evidence

![Completed backend deployment steps in GitHub Actions](/images/learnsphere-backend-deploy-job.png)

*Figure 5.41. The backend job completes testing, OIDC-based AWS authentication, ECR login, Docker image build and push, and the Auto Scaling Group rollout.*

![Immutable Docker images stored in Amazon ECR](/images/learnsphere-cicd-ecr-images.png)

*Figure 5.42. The `learnsphere-be-2` repository stores images tagged by commit SHA, enabling release traceability and rollback to a previous artifact.*
