---
title: "MongoDB Atlas and shared data"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.8.1. </b> "
---

#### 1. MongoDB Atlas role

The backend uses Mongoose document schemas for:

* Accounts, roles, and profiles.
* Courses, lessons, and enrollments.
* Quizzes, questions, attempts, and results.
* Notifications, upload sessions, and AI rate-limit buckets.
* AI messages, indexed text, summaries, and token usage.

The database is independent of EC2 lifecycle. ASG replacement does not remove business data, and every new instance connects to the same cluster.

#### 2. Connectivity from private subnets

```text
EC2 private subnet 1a → NAT Gateway 1a ┐
                                      ├→ MongoDB Atlas TLS
EC2 private subnet 1b → NAT Gateway 1b ┘
```

Atlas Network Access allowlists the two NAT Elastic IPs as `/32`, not the dynamic private address of each EC2 instance. The connection string is stored inside `/learnsphere/prod/backend-env` as a SecureString.

The readiness endpoint reports ready only when the database is connected:

```json
{
  "status": "ready",
  "database": "connected"
}
```

#### 3. Consistency under HA

Two instances can serve requests concurrently because authentication state is carried in cookies/JWT and durable state is stored in MongoDB/S3. Enrollment, attempts, and AI history are never kept only in one instance's memory.

`MONGODB_REQUIRE_TRANSACTIONS` can require transactions for multi-step production operations when supported by the cluster. Unique indexes and atomic updates protect races such as AI rate limiting and summary generation.

#### 4. Current choice and future direction

MongoDB Atlas matches the current Mongoose schemas and completed business workflows. An immediate DynamoDB migration would require redesigning partition keys, access patterns, transactions, and data migration.

At a larger scale, DynamoDB can be evaluated per workload where access patterns are predictable or serverless scaling is valuable; it should not mechanically replace every MongoDB collection. Other improvements include Atlas Private Endpoint/peering, backup policy, multi-region data, and restore testing.

#### Deployment evidence

![MongoDB Atlas IP Access List restricted to the two NAT Gateway public IP addresses](/images/learnsphere-mongodb-nat-ip-access-list.png)

**Figure 5.47:** MongoDB Atlas Network Access allows only the two `/32` public IP addresses of the NAT Gateways in `ap-southeast-1a` and `ap-southeast-1b`. Both entries are active, while connection strings and user information remain hidden.

![MongoDB Atlas collections used by the LearnSphere backend](/images/learnsphere-mongodb-collections.png)

**Figure 5.48:** Atlas Collections shows the shared persistent data model used by both backend instances, including courses, lessons, enrollments, lesson progress, quiz attempts, notifications, AI messages, and AI rate-limit buckets. No individual document containing personal data is opened.
