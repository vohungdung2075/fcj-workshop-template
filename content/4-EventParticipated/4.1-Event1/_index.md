---
title: "Event 1"
date: 2026-06-13
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Event Summary Report: “Meetup 13/06/2026”

### Event Objectives

- **Explore Cloud Architecture Design on AWS**: Learn how to build highly available, scalable, and low-latency URL shortening services.
- **Understand MNC Work Culture & Career Mindset**: Equip critical thinking, Data Storytelling skills, and personal career development frameworks.
- **Demystify the Role of a DevOps Engineer**: Clear common misconceptions about DevOps, establish learning roadmaps and practical system thinking.
- **Career Pathways from Student to AWS Specialist**: Understand the 8-stage growth journey from FCAJ student to specialist at AWS Partner organizations.

---

### Key Speakers & Topics

1. **Dinh Trung Kien** *(Lead Developer at Startup)* & **Nguyen Minh Tho** *(Student)*
   - **Topic:** *A scalable URL shortening service on AWS*
2. **Mr. Dat Pham** *(Data Analytics Engineer)* & **Mr. Cuong Nguyen** *(Process Engineer)*
   - **Topic:** *Real-world stories to MNC corporate culture*
3. **Truong Huu Trong** *(DevOps Engineer @ Endava Vietnam)*
   - **Topic:** *What does a DevOps Engineer really do?*
4. **Danh Hoang Hieu Nghi** *(AI Engineer – AWS Community Builder – AWS Student Builder Group Leader)*
   - **Topic:** *From First Cloud AI Journey to AWS Partner*

---

### Key Highlights

#### 1. A Scalable URL Shortening Service on AWS (Dinh Trung Kien & Nguyen Minh Tho)
- **Problem Statement**: Standard flows (User → FE → BE → DB) are easy to deploy but face security vulnerabilities, high read latency, Single Point of Failure (SPOF), and scaling bottlenecks under peak loads.
- **AWS Cloud Architecture**: Utilizes Route 53, CloudFront CDN, and WAF at the edge; Backend deployed containerized on Multi-AZ **AWS Fargate (ECS)** behind an Application Load Balancer (ALB).
- **Key Generation Service (KGS)**: Background service running on ECS to pre-generate random unique short codes into a Redis queue (`LPUSH/RPOP`), enabling instant URL creation without collision.
- **Cache-aside Pattern**: Combines **Amazon DynamoDB** and **Amazon ElastiCache for Redis**; reads hit Redis first (*Cache Hit*), consulting DynamoDB only on *Cache Miss* to minimize database load.

#### 2. Real-World Stories to MNC Corporate Culture (Mr. Dat Pham & Mr. Cuong Nguyen)
- **Data Analytics Engineering**: Operational GMV tracking at Kamereo vs. IoT factory machine data optimization at Colgate-Palmolive.
- **Core 4 Skillsets**: Critical Thinking (*Psychology*), Communication (*Forum*), Data Storytelling, and Problem Solving.
- **Career Growth Model**: 5-stage progression framework (`Follower` → `Learner` → `Problem Solver` → `System Thinker` → `Super Star`).
- **MNC Culture**: 4-stage hiring workflow (ATS, Test, STAR interview, Cultural fit) and *No-Blame Post-Mortem* mindset (root cause resolution over individual blame).

#### 3. What Does a DevOps Engineer Really Do? (Truong Huu Trong - Endava Vietnam)
- **Demystifying DevOps**: Debunked "only writing CI/CD or midnight bug fixing"; emphasized that scope depends on project scale, product complexity, and infrastructure maturity.
- **Daily Operations**: Resolving environment downtime, cloud bill spikes, security vulnerabilities, and driving Dev-Ops alignment (On-call, Incident postmortem, Cost investigation).
- **Learning Roadmap**: Master fundamentals (Linux, Networking, Python/Golang, Git, Containers) → Understand app runtime → Build small projects (break & fix).
- **Key Principles**: Ask "Why" before "How", focus on System Thinking over quick fixes, and leverage AI to boost productivity.

#### 4. From First Cloud AI Journey to AWS Partner (Danh Hoang Hieu Nghi)
- **Growth Roadmap**: From initial curiosity (`Student Curiosity`) → community participation (`First Cloud Journey`) → project building (`Portfolio`) → professional specialist (`AWS Partner`) → sharing back (`Share Back`).
- **Career Path**: Shared growth experience across Solutions Architect, DevOps, AI Engineer roles and career opportunities at AWS Partners like Renova Cloud (*AWS Partner of the Year 2026*).

---

### Key Takeaways

#### System Design Mindset
- **Separation of Concerns**: Fully separating read and write paths to optimize each for its own traffic pattern rather than sharing a single bottleneck.
- **Pre-computation over On-demand**: Pre-generating short codes ahead of time using Key Generation Service (KGS) so creation requests are instant and collision-free under heavy traffic.
- **System Thinking vs. Quick Fixes**: Evolving from a checklist-driven executor (*Follower*) to a holistic *System Thinker*, assessing long-term architectural impacts over temporary patches.

#### Technical & Cloud Architecture
- **Cache-aside Pattern**: Serving reads from in-memory Redis cache first and consulting DynamoDB only on cache misses, keeping latency minimal and main DB stress low.
- **Defense at the Edge**: Pushing security (AWS WAF) and caching (CloudFront CDN) as close to the user as possible, stopping threats before they reach core infrastructure.
- **Compute Spectrum**: Selecting appropriate compute options (AWS ECS Fargate Multi-AZ) to balance serverless management convenience with dynamic auto-scaling.

#### Career Growth & Corporate Culture
- **Data Storytelling**: Transforming raw operational metrics (GMV, Fulfillment Cost, Fill Rate) into compelling narratives that drive business decisions.
- **No-Blame Post-Mortem Culture**: Focusing on root cause analysis and systemic improvements after production incidents rather than placing individual blame.
- **Progression Roadmap**: Advancing from initial curiosity (`Student Curiosity`) → hands-on projects (`Portfolio`) → industry expert (`AWS Partner`).

---

### Applying to Work & LearnSphere Project

- **Apply Cache-aside & Presigned URLs**: Integrate ElastiCache Redis and S3 Presigned URLs into the **LearnSphere** project to accelerate video streaming and document delivery, minimizing MongoDB database queries.
- **Standardize Containerization & CI/CD**: Implement optimized Dockerfiles and GitHub Actions pipelines for automated Express.js backend deployment.
- **Enhance Logging & Observability**: Set up CloudWatch Logs and automated alarms targeting core application workflows (authentication, quiz submissions) rather than static server CPU metrics.
- **Adopt Data Storytelling in Technical Reports**: Utilize structured diagrams and metric visualizations to present the FCJ internship report professionally.
- **Practice No-Blame Code Reviews**: Establish rigorous PR review workflows focused on systemic reliability and code quality enhancement.

---

### Event Experience & Reflection

Attending **Meetup 13/06/2026** was an engaging and inspiring learning journey, offering multi-dimensional perspectives from deep technical architecture to career growth strategies. Key highlights include:

#### Learning from Practitioner Speakers
- Gained authentic insights into the daily work of Data Analytics Engineers and DevOps Engineers at major organizations (Colgate-Palmolive, Endava Vietnam). Real-world case studies helped bridge the gap between academic theory and enterprise operations.
- Deeply inspired by the 5-stage career growth model (advancing from *Follower* to *Problem Solver* and *System Thinker*), setting clear milestones for my professional journey.

#### Rich Technical Content
- Deep-dived into vivid case studies: from solving high-throughput URL shortening via Key Generation Service & Redis Cache to supply chain operations and manufacturing data analytics.
- Speakers clearly illustrated the "behind-the-scenes" yet critical role of DevOps, providing a clear roadmap across modern cloud toolchains.

#### Community Networking & Expanding Horizons
- Gained deeper understanding of student technology communities like **AWS Student Builder Group** and the **First Cloud AI Journey (FCAJ)** program. The speaker's real-world path from student to AWS Partner provided immense drive to proactively seek learning, project building, and networking opportunities.

#### Key Takeaways
- Maintain a *Learner mindset*, staying curious and asking "Why" before "How" on any technical implementation.
- Build a well-rounded skill portfolio combining technical fundamentals (Linux, Cloud, Caching), soft skills (Data Storytelling, communication), and corporate culture integration (*No-blame culture*).

#### Event Photos

![Commemorative group photo with the AWS Student Builders community at AWS Vietnam Office](/images/events/event-1/event1_group.jpg)

![Check-in at the Meetup event on 26th Floor Bitexco Tower](/images/events/event-1/event1_selfie.jpg)

> **Summary:** The event provided not only valuable Cloud & DevOps knowledge but also expanded career horizons, building strong confidence for my internship at AWS Bootcamp.
