---
title: "Resource cleanup"
date: 2026-07-30
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

Cleanup is a controlled decommissioning stage performed after project evaluation. Its purpose is not merely to delete resources, but also to preserve required data, avoid disrupting shared infrastructure, and verify that no forgotten service continues to incur charges.


#### 1. Define the cleanup scope

Before deleting anything, inventory every LearnSphere resource by Region and service:

* CloudFront distribution, CloudFront Function, and Origin Access Control.
* The `learnsphere-fe-2` and `learnsphere-media-2` S3 buckets.
* Application Load Balancer, HTTPS listener, and Backend Target Group.
* Auto Scaling Group, Launch Template, and ASG-managed EC2 instances.
* The `learnsphere-be-2` ECR repository and Docker images.
* Production VPC, four subnets, Internet Gateway, two NAT Gateways, two Elastic IPs, route tables, and S3 Gateway Endpoint.
* Parameter Store, CloudWatch log groups and alarms, and the SNS topic.
* GitHub OIDC deployment role, EC2 instance profile, and related policies.
* ACM certificates in `us-east-1` for CloudFront and `ap-southeast-1` for the ALB.
* DNS records at the domain provider, MongoDB Atlas resources, and the Groq API key.

Verify names, `Project=LearnSphere` tags, ARNs, and Regions before every deletion. Do not remove service-linked roles, OIDC providers, or shared resources while another workload still depends on them.

#### 2. Back up data and evidence

Create and verify backups before changing traffic or infrastructure:

1. Export the required MongoDB Atlas collections and record the database, cluster, and IP Access List configuration.
2. Back up important S3 Media objects, including videos, documents, thumbnails, and avatars.
3. Preserve the source code, Dockerfile, GitHub Actions workflow, and the final stable commit.
4. Record the VPC, route table, Security Group, ALB, Target Group, ASG, Launch Template, and CloudFront configuration.
5. Preserve Parameter Store names and environment-variable structure without publishing SecureString values.
6. Export logs, metrics, test results, and cost evidence required by the workshop report.
7. Confirm that the backups can be opened or imported before deletion begins.

#### 3. Prevent new deployments

Stop the pipeline from recreating resources or starting another Instance Refresh:

1. Disable the workflow's `push` trigger or temporarily disable GitHub Actions for the repository.
2. Confirm that no workflow run, ASG Instance Refresh, or scaling activity is still running.
3. Keep GitHub Secrets and IAM roles temporarily because they may still be required to verify or complete the cleanup.
4. Record the final production image tag stored in `/learnsphere/prod/backend-image-tag`.

#### 4. Stop the highly available Backend

Stop the Backend through its orchestration layer. Do not terminate individual instances manually because the ASG will replace them.

1. Disable scheduled actions and scaling policies if present.
2. Set the ASG to `min=0` and `desired=0`; `max=4` may remain while instances terminate.
3. Monitor **Instance management** until no instance remains `InService` or `Terminating`.
4. Confirm that the Target Group no longer contains registered targets.
5. Delete the Auto Scaling Group after capacity reaches zero.
6. Delete all Launch Template versions used only by LearnSphere.
7. Verify that no Backend EC2 instance or related network interface remains.

The `/health/ready` endpoint and application APIs become unavailable after this stage. This is expected during decommissioning.

#### 5. Delete the load-balancing layer

Perform these steps after the ASG has stopped so that targets cannot be registered again:

1. Delete the HTTPS listener and listener rules.
2. Delete `LearnSphere-Prod-ALB`.
3. Wait for the ALB and its network interfaces to disappear.
4. Delete `LearnSphere-Backend-TG`.
5. Delete `LearnSphere-ALB-SG` and `LearnSphere-Backend-SG` only after no resource references them.

If AWS reports that a resource is still in use, inspect the ASG–Target Group integration, listeners, network interfaces, and Security Group references rather than forcing an incorrect deletion order.

#### 6. Remove VPC networking in dependency order

NAT Gateways should be prioritized because hourly and data-processing charges continue until they are deleted.

1. Delete the two NAT Gateways in public subnets 1a and 1b.
2. Wait for the `Deleted` state, then release both associated Elastic IP addresses.
3. Delete the S3 Gateway Endpoint when no workload requires it.
4. Remove each private route table's `0.0.0.0/0` route to its NAT Gateway.
5. Remove subnet associations and delete custom route tables.
6. Delete both private and both public subnets.
7. Delete the remaining custom Security Groups and Network ACLs.
8. Detach the Internet Gateway from the VPC before deleting it.
9. Delete the production VPC after all subnets, endpoints, gateways, ENIs, and dependent resources are gone.

For a temporary shutdown rather than permanent removal, the VPC and subnets may be retained, but NAT Gateways should still be removed to avoid fixed hourly charges.

#### 7. Remove Frontend, CloudFront, and S3 resources

CloudFront must be removed before deleting origin access configuration and buckets:

1. Remove the custom domain from CloudFront if it will be assigned to another system.
2. Disable the distribution and wait until the change is fully deployed.
3. Delete the distribution, CloudFront Function, and LearnSphere-specific Origin Access Control.
4. Delete `index.html`, `assets/`, and all remaining objects from the Frontend bucket.
5. Delete S3 Media objects only after validating the backup.
6. If versioning is enabled, delete all object versions and delete markers.
7. Abort incomplete multipart uploads so that they no longer consume storage.
8. Remove bucket policies and CORS rules, then delete both buckets.

Deleting S3 Media before the database would leave database keys pointing to missing objects. Therefore, verify that the application has stopped and backups are complete.

#### 8. Remove images, configuration, and observability

After no EC2 instance needs to boot from the production image:

1. Delete ECR images and then delete `learnsphere-be-2`.
2. Delete `/learnsphere/prod/backend-image-tag` and `/learnsphere/prod/backend-env`.
3. Delete Backend-specific CloudWatch alarms, log streams, and log groups.
4. Export or retain operational logs first when they are required for auditing.
5. Delete the SNS subscription before deleting `LearnSphere-Alerts2`.
6. Check for CloudWatch dashboards and metric filters that still reference removed resources.

Parameter Store values are deleted after the Backend so that an ASG capable of creating instances never loses its bootstrap configuration prematurely.

#### 9. Revoke IAM, certificates, and DNS

Complete the security cleanup after workloads have stopped:

1. Remove LearnSphere-specific inline and managed deployment policies.
2. Delete the GitHub deployment role and EC2 role/instance profile after no resource uses them.
3. Delete the GitHub OIDC provider only when no other repository or project uses it.
4. Remove obsolete GitHub Secrets such as role ARN, bucket name, CloudFront distribution ID, and API base URL.
5. Delete the `www.learnspherev2.id.vn` ACM certificate in `us-east-1` after CloudFront deletion.
6. Delete the `origin.learnspherev2.id.vn` ACM certificate in `ap-southeast-1` after ALB deletion.
7. Remove obsolete `www`, `origin`, and ACM validation records from TenTen.
8. Retain the domain for the portfolio or report, or disable automatic renewal when it is no longer required.

#### 10. Handle MongoDB Atlas and Groq

MongoDB Atlas and Groq are external to AWS and must be reviewed separately:

* Remove both NAT Gateway Elastic IPs from the MongoDB Atlas IP Access List.
* Pause or delete the cluster only after exporting data; delete the Atlas project only if it contains no other database.
* Revoke or rotate the production Groq API key.
* Remove local environment secrets and `.env` files from devices no longer used.
* Never include database URIs, API keys, or credentials in cleanup documentation.

#### 11. Post-cleanup verification

Finish with an independent verification pass:

* Search for `LearnSphere` through Resource Groups, Tag Editor, and each service console in `ap-southeast-1`.
* Review global services and other Regions, especially CloudFront, IAM, and ACM in `us-east-1`.
* Confirm that no unintended EC2, ASG, ALB, NAT Gateway, Elastic IP, ECR, or S3 resource remains.
* Verify that DNS no longer targets deleted CloudFront or ALB resources.
* Confirm that GitHub Actions can no longer deploy through the retired role.
* Monitor Billing and Cost Explorer for 24–48 hours because cost data is delayed.
* Record the cleanup time, operator, retained resources, and deleted resources.

LearnSphere is considered safely decommissioned when required data is preserved, credentials are revoked, and no abandoned production infrastructure continues to generate cost.
