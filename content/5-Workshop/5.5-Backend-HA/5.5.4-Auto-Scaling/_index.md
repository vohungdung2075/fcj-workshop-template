---
title: "Multi-AZ Auto Scaling Group"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
---

#### Objective

The Auto Scaling Group maintains Backend capacity, distributes instances across two Availability Zones, and replaces unhealthy instances. It removes the compute-tier single point of failure.

#### 1. Create the Auto Scaling Group

Create `LearnSphere-Backend-ASG`:

| Setting | Value |
| --- | --- |
| Launch Template | `LearnSphere-Backend-LT`, Default version |
| Private subnet 1a | `subnet-04724edeb47832763` |
| Private subnet 1b | `subnet-0ccb5a5e29560fa75` |
| Target Group | `LearnSphere-Backend-TG` |
| Minimum capacity | 2 |
| Desired capacity | 2 |
| Maximum capacity | 4 |

With desired capacity 2 and two private subnets, the ASG maintains one Backend instance per AZ when capacity and infrastructure allow.

#### 2. Configure health checks

| Attribute | Value |
| --- | --- |
| Health check types | EC2 and ELB |
| Health check grace period | 360 seconds |
| Default instance warmup | 300 seconds |
| Default cooldown | 300 seconds |

EC2 health detects virtual-machine failures. ELB health detects a missing container, closed port 5000, or failed readiness. Combining both prevents the ASG from retaining a `Running` instance whose application cannot serve requests.

#### 3. Configure instance maintenance

| Attribute | Value |
| --- | --- |
| Replacement behavior | Launch before terminating |
| Minimum healthy percentage | 100% |
| Maximum healthy percentage | 200% |

The ASG may temporarily run additional capacity during replacement. An old instance is removed only after a new instance boots and becomes healthy in the Target Group.

#### 4. Service-linked role

The ASG uses:

```text
AWSServiceRoleForAutoScaling
```

The first creation attempt may report that the service-linked role is not ready. This is a short propagation delay while AWS creates the role; retry after it becomes available.

#### 5. Capacity versus scaling

`minimum=2` prevents the group from scaling below two Backends in normal operation. `desired=2` is the current operating capacity. `maximum=4` is only the upper boundary.

Without a dynamic scaling policy, the ASG does not automatically grow from two to three or four instances based on CPU. Future Target Tracking can use:

* ALB RequestCountPerTarget.
* Average CPUUtilization.
* A custom response-time or queue metric.

#### 6. Validate the ASG

Capacity Overview must show:

```text
Desired capacity: 2
Minimum capacity: 2
Maximum capacity: 4
Healthy: 2/2
```

Instance management must show two `InService`, `Healthy` instances using Launch Template version 2 in different Availability Zones.

#### Deployment evidence

![LearnSphere Backend Auto Scaling Group capacity](/images/learnsphere-asg-capacity.png)

*Figure 5.23. `LearnSphere-Backend-ASG` is at desired capacity with two instances and scaling limits of 2–4. It uses `LearnSphere-Backend-LT`, preserving a minimum of two Backend instances during normal operation.*

![Healthy LearnSphere Backend instances distributed across two Availability Zones](/images/learnsphere-asg-instances-multiaz.png)

*Figure 5.24. Instance management reports two `InService`, `Healthy` `t3.small` instances. They are distributed between `apse1-az2` (`ap-southeast-1a`) and `apse1-az1` (`ap-southeast-1b`), confirming Multi-AZ placement.*
