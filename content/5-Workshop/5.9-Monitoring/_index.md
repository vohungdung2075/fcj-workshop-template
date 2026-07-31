---
title: "Monitoring and alerting"
date: 2026-07-30
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

#### Monitoring objectives

Production monitoring must answer four operational questions:

1. Is the application reachable through CloudFront and the ALB?
2. Are both backend targets ready to serve requests?
3. Is the application producing errors, slow responses, or AI-provider failures?
4. Can the platform recover when an EC2 instance becomes unhealthy?

The monitoring boundary therefore covers CloudFront, ALB, Target Group, Auto Scaling Group, EC2 containers, MongoDB readiness, and external AI calls. CloudWatch centralizes AWS metrics and application logs, while SNS delivers alarm notifications to the administrator.

| Signal source | Main evidence | Operational purpose |
| --- | --- | --- |
| ALB/Target Group | target health, response time, HTTP 5xx | Detect unavailable or degraded backends |
| Auto Scaling Group | desired, in-service, pending instances | Confirm capacity and self-healing |
| EC2/Docker | application stdout/stderr | Diagnose request and runtime failures |
| `/health/ready` | application and database readiness | Prevent traffic from reaching an unready target |
| AI integration | normalized error codes in logs | Distinguish quota, timeout, credential, and provider failures |

#### Centralized CloudWatch Logs

Docker uses the `awslogs` logging driver to send logs from every EC2 instance to:

```text
/learnsphere/backend2
```

Each instance/container has a separate log stream. Logs remain available in one place when the ASG replaces an instance, so troubleshooting does not depend on SSH access or a surviving local disk. A retention period should be configured to control storage cost and comply with the project’s data-retention policy.

Example query for recent application and AI failures:

```sql
fields @timestamp, @message
| filter @message like /ERROR|Exception|AI_THROTTLED/
| sort @timestamp desc
| limit 100
```

Logs must not contain JWT values, cookies, MongoDB connection strings, email credentials, Groq keys, full prompts, or private presigned URLs. Identifiers should be masked when they are not necessary for troubleshooting.

#### Health and replacement chain

The application exposes a liveness endpoint for process availability and a readiness endpoint for serving production traffic. `/health/ready` verifies both the application and its database dependency:

```json
{"status":"ready","database":"connected"}
```

The ALB Target Group calls the readiness endpoint on port `5000`. Only healthy targets receive traffic. The ASG also uses ELB health status, so a repeatedly failing instance is removed and recreated from the default Launch Template. This creates the following recovery chain:

```text
Readiness fails
→ Target Group marks the target unhealthy
→ ALB stops forwarding new requests to that target
→ ASG replaces the unhealthy instance
→ User data installs Docker and starts the current image
→ Target becomes healthy and returns to service
```

#### CloudWatch Alarms and SNS

SNS topic `LearnSphere-Alerts` sends email notifications to the administrator. In the final ASG architecture, alarms must follow logical resources instead of fixed instance IDs:

| Metric | Initial condition | Meaning |
| --- | --- | --- |
| ALB `UnHealthyHostCount` | `>= 1` for two periods | At least one backend target is unavailable |
| ALB `HTTPCode_Target_5XX_Count` | exceeds the accepted error baseline | Application or dependency failures |
| ASG `GroupInServiceInstances` | `< 2` | HA capacity is below the required minimum |
| ALB `TargetResponseTime` | remains above the agreed latency threshold | Backend is degraded even if still healthy |
| CloudWatch Logs metric filter | `ERROR`, `Exception`, or repeated AI failures | Application-level incident requiring investigation |

Legacy `CPUUtilization` or `StatusCheckFailed` alarms attached to old instance IDs do not represent instances created later by the ASG. They should be removed or replaced with ASG/ALB alarms, per-instance dynamic monitoring, or a composite alarm.

#### Incident response runbook

When an alarm enters `ALARM`, the operator should:

1. Confirm the affected resource and time window from the alarm history.
2. Inspect Target Group health and ASG activity before changing capacity manually.
3. Query `/learnsphere/backend2` for the same period and correlate errors with the affected log stream.
4. Check `/health/ready` and distinguish application, database, network, and external-provider failures.
5. Allow the ASG replacement process to complete when the failure is instance-specific.
6. Record the cause, recovery action, and prevention task after service returns to normal.

Useful read-only checks:

```bash
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names LearnSphere-Backend-ASG

aws elbv2 describe-target-health \
  --target-group-arn <TARGET_GROUP_ARN>

aws logs tail /learnsphere/backend2 --since 15m
```

#### Deployment evidence

![Backend CloudWatch log streams](/images/learnsphere-cloudwatch-log-streams.png)

*Figure 5.56. CloudWatch log streams produced across multiple backend deployments and instance-replacement cycles. The `asg-` stream prefix makes it possible to correlate application logs with each EC2 private hostname without signing in to the server.*

![CloudWatch alarm states](/images/learnsphere-cloudwatch-alarms.png)

*Figure 5.57a. CloudWatch Alarms at the time of validation, including Auto Scaling target-tracking alarms and the legacy EC2 alarms. A target-tracking Low alarm may enter the `In alarm` state when demand is low so that scale-in can occur; it does not by itself indicate an application outage. The fixed-instance alarms remain as evidence of the earlier deployment stage and should be replaced by ALB/ASG-scoped alarms.*

![Confirmed SNS subscription](/images/learnsphere-sns-confirmed-subscription.png)

*Figure 5.57b. The `LearnSphere-Alerts2` SNS topic has a confirmed email subscription and is ready to receive notifications emitted by CloudWatch Alarms.*
