---
title: "Release validation and rollback"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.7.4. </b> "
---

#### 1. Validate the backend after refresh

The workflow checks:

```text
ASG desired capacity == InService instance count
ASG healthy capacity == desired capacity
Instance Refresh == Successful
GET https://origin.learnspherev2.id.vn/health/ready == HTTP 200
```

The readiness endpoint also confirms that MongoDB is connected, so a running container without database access is not considered a successful release.

#### 2. Rollback mechanism

Before writing a new tag, the pipeline saves `OLD_IMAGE_TAG`. If refresh or public health validation fails:

1. Restore the old tag in Parameter Store.
2. Start another Instance Refresh.
3. New EC2 instances bootstrap from the last known-good image.
4. Mark the workflow as failed for investigation.

Rollback follows the same deployment path as a normal release and never relies on SSH changes to individual instances.

#### 3. Validate the complete workflow

After a successful run, verify:

| Check | Result |
| --- | --- |
| Backend tests | Pass |
| ECR image tag | Matches commit SHA |
| Instance Refresh | Successful |
| Target Group | 2 healthy, 0 unhealthy |
| Origin readiness | HTTP 200 |
| Frontend build | Success |
| S3 publish | Success |
| CloudFront invalidation | Completed |

#### Deployment evidence

![Both LearnSphere deployment jobs completed in GitHub Actions](/images/learnsphere-github-actions-overview.png)

*Figure 5.45. The production workflow completed the backend and frontend jobs in sequence. Both green jobs provide aggregate evidence, but health checks and AWS resource state are still required.*


![Successful Instance Refresh history for the backend ASG](/images/learnsphere-asg-instance-refresh-history.png)

*Figure 5.46. The Auto Scaling history shows three Instance Refresh operations in the `Successful` state at 100% completion, confirming stable instance replacement during rollout.*

There is no need to cause a production failure for a rollback screenshot. The installed rollback branch can be demonstrated through a diagram or code snippet.
