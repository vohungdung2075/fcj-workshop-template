---
title: "CI/CD automation with GitHub Actions"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Objectives

LearnSphere uses `.github/workflows/deploy.yml` to validate and release the backend before publishing the frontend. The pipeline signs in to AWS through GitHub OIDC and a short-lived IAM role; no long-lived access key is stored in the repository.

Expected results:

* Every commit is validated before release.
* The backend Docker image is tagged with the exact Git commit SHA and stored in ECR.
* The backend rolls out through ASG Instance Refresh with at least 100% healthy capacity.
* A failed rollout restores the previous image tag.
* The frontend deploys only after the backend succeeds.
* S3 cache metadata and CloudFront invalidation are automated.
* Two production deployments cannot overlap.

#### Workflow structure

```text
push main / workflow_dispatch
→ Backend: test → build → ECR → Parameter Store → Instance Refresh → health
→ Frontend: build → S3 sync → index.html → CloudFront invalidation
```

The workflow uses `concurrency: learnsphere-production` with `cancel-in-progress: false`. An active instance replacement is never cancelled halfway through, and the next release must wait.

#### Deployment steps

1. [Configure the workflow, OIDC, and least-privilege access](5.7.1-workflow-oidc/)
2. [Build and release the backend through ECR and ASG](5.7.2-backend-pipeline/)
3. [Build and release the frontend through S3 and CloudFront](5.7.3-frontend-pipeline/)
4. [Validate releases and rollback](5.7.4-validation-rollback/)

#### Completion criteria

CI/CD is complete when both jobs are green, ECR contains the commit-SHA image, the ASG has healthy instances in both AZs, the public readiness endpoint returns HTTP 200, the new frontend is visible on the custom domain, and CloudFront invalidation completes.
