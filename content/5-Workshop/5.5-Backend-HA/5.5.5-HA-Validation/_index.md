---
title: "High Availability and self-healing validation"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5.5.5. </b> "
---

#### Objective

Two EC2 icons in an architecture diagram do not prove High Availability. Validation must cover the Target Group, ASG, readiness endpoint, and instance replacement history.

#### 1. Validate the Target Group

Both targets in `LearnSphere-Backend-TG` must:

* Be `Healthy`.
* Reside in different Availability Zones.
* Listen on port 5000.
* Leave no `Unused`, `Initial`, `Draining`, or `Unhealthy` target after stabilization.

The Target Group sends an HTTP health check to `/health/ready`, validating the application rather than only checking whether EC2 is powered on.

#### 2. Validate public readiness

```powershell
curl.exe -i https://origin.learnspherev2.id.vn/health/ready
```

Expected result:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

```json
{"status":"ready","database":"connected"}
```

This validates DNS → HTTPS listener → ALB → Target Group → Backend → MongoDB.

#### 3. Validate each instance

Through Systems Manager Run Command:

```bash
sudo docker ps --filter name=learnsphere-be
sudo docker inspect \
  --format 'status={{.State.Status}} restarts={{.RestartCount}} image={{.Config.Image}}' \
  learnsphere-be
curl -fsS http://127.0.0.1:5000/health/ready
```

Run this on both instances. Both containers must use the same production image tag and report `database=connected`.

#### 4. Validate self-healing

Use Instance Refresh history or a controlled test:

1. Confirm that both targets are healthy.
2. Terminate one ASG instance through Auto Scaling or EC2.
3. Observe the ALB remove the old target from service.
4. Observe the ASG launch a replacement from the Launch Template.
5. The new instance bootstraps from Parameter Store and ECR.
6. The new target moves from `Initial` to `Healthy`.
7. The remaining target continues serving the endpoint during replacement.

Only perform the termination test when both initial targets are healthy and enough observation time is available. Do not reduce desired capacity during a self-healing test.

#### 5. Validate launch-before-terminate

The pipeline starts Instance Refresh with:

```text
MinHealthyPercentage: 100
MaxHealthyPercentage: 200
InstanceWarmup: 300
```

The workflow:

1. Confirms that the candidate image exists in ECR.
2. Verifies that the ASG is healthy and no other refresh is active.
3. Records the previous image tag.
4. Publishes the candidate tag to Parameter Store.
5. Starts Instance Refresh.
6. Waits for successful refresh and healthy InService capacity.
7. Calls the readiness endpoint through the ALB.
8. Restores the old tag and starts a rollback refresh after failure.

#### 6. HA scope

The architecture provides Backend HA within one AWS Region across two Availability Zones. It tolerates a failed container or EC2 instance and retains service while one target is removed.

It is not a multi-Region Disaster Recovery design. MongoDB Atlas and Groq remain external dependencies, so total availability also depends on the Atlas cluster configuration and Groq service.

#### Validation evidence

![LearnSphere Backend Target Group with two healthy targets](/images/learnsphere-target-group-healthy.png)

*Figure 5.25. `LearnSphere-Backend-TG` reports two total targets, two healthy targets, and zero unhealthy targets on HTTP port 5000. Combined with the Multi-AZ placement in Figure 5.24, this confirms that both Backend instances are eligible to receive ALB traffic.*

![LearnSphere readiness endpoint returns HTTP 200](/images/learnsphere-readiness-200.png)

*Figure 5.26. Calling `https://origin.learnspherev2.id.vn/health/ready` through the public ALB returns HTTP 200 with `{"status":"ready","database":"connected"}`, validating the complete TLS, load-balancing, Backend, and database path.*

![Successful LearnSphere Auto Scaling Group replacement activity](/images/learnsphere-asg-self-healing.png)

*Figure 5.27. ASG Activity history records successful instance termination during an Instance Refresh while the group returns to desired capacity with 2/2 healthy instances. This provides operational evidence of controlled replacement and self-healing behavior.*
