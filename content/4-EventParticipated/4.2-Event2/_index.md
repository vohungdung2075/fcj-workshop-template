---
title: "Event 2"
date: 2026-07-11
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Event Summary Report: “Meetup 11/07/2026”

### Event Objectives

- **Witness the Cloud Architect Competition Finals**: Cheer on the top 2 teams competing through 10 challenging AWS Cloud architecture questions.
- **Explore Web App Security Automation (AWS Security Agent)**: Learn how Agentic AI powered by Amazon Bedrock automates pentesting, code security reviews, and architecture reviews.
- **Master SLA & True Monitoring Practices**: Embrace the Monitoring Pyramid framework, distinguishing "Healthy Infrastructure" from "Healthy User Experience".
- **Conquer the AWS Certified Cloud Practitioner Exam (CLF-C02)**: Understand exam structure, 4 domain weightings, Keyword Mapping strategies, and preparation roadmaps.

---

### Event Agenda & Speakers

- **Opening Activity:** *Cloud Architect Competition Finals* (Intense matchup between top 2 teams answering 10 AWS Cloud scenario questions).
- **Session 1:** **Thinh Nguyen** *(DevOps/DevSecOps/Cloud Engineer @ Styl Solutions, FCAJ)*
  - **Topic:** *Securing Your Web Apps With AWS Security Agent*
- **Session 2:** **Nguyen Huynh Son** *(Infrastructure Support Engineer @ Endava, Ex-SPS, AWS Student Builder Group HUFLIT)*
  - **Topic:** *SLA and Monitoring - From SLA to Monitoring what really matters*
- **Session 3:** **Ngo Le Tan Huy** *(Cloud Practitioner Presenter)*
  - **Topic:** *Inside The Exam: AWS Cloud Practitioner (CLF-C02)*

---

### Key Highlights

#### Opening Activity: Cloud Architect Competition Finals
- **Format**: The top 2 finalist teams (KLKAT vs Ngu Dai Hiep) competed directly on stage, tackling **10 scenario questions covering AWS Cloud infrastructure** (VPC, IAM, EC2, S3, High Availability, Security).
- **Results & Atmosphere**: The matchup was highly intense with close scores. The team with faster, accurate responses and higher total points earned the championship title.

#### 1. Securing Your Web Apps With AWS Security Agent (Thinh Nguyen - Styl Solutions)
- **Security Bottleneck**: Manual pentesting takes weeks, costs $5k-$20k per audit, and heavily relies on human pentester availability.
- **AWS Security Agent (Frontier Agent)**: Powered by Amazon Bedrock, covering full lifecycle from Design Review (Markdown/Terraform), Code Security Review (GitHub/GitLab PR integration with auto-fix patches), to Automated Pentesting (IDOR -> XSS exploit chains with attack graphs).
- **Pricing & Limitations**: Pay-as-you-go at $5/Task-Hour (significant savings over human pentest teams); limited by MFA/biometric blocks and complex business logic flaws.

#### 2. SLA and Monitoring - From SLA to Monitoring What Really Matters (Nguyen Huynh Son - Endava)
- **SLA & Risk Management**: SLA (Service Level Agreement) sets service expectations. Monitoring forms a core part of risk management (Identify -> Monitor -> Respond -> Improve).
- **Core Message**: **"Healthy Infrastructure ≠ Healthy User Experience"**. 100% green infrastructure metrics (CPU/Memory) do not guarantee customer satisfaction. Case study: `/api` returns `200 OK` while `/login` fails due to RDS DB connection errors, dropping login success to 0%.
- **Monitoring Pyramid & Alerting**: Building a monitoring hierarchy from Cloud Provider → Infrastructure → Application → Business → Customer Experience. Setting automated alert pipelines: Custom Metric → CloudWatch Alarm → SNS Topic → Email/Slack.

#### 3. Inside The Exam: AWS Cloud Practitioner CLF-C02 (Ngo Le Tan Huy)
- **Exam Overview**: Foundational exam with 65 multiple-choice questions (90 mins, score range 100-1000, passing score 700, 3-year validity).
- **4 Exam Domains**: Domain 1: Cloud Concepts (24%), Domain 2: Security & Compliance (30% - Shared Responsibility Model, IAM), Domain 3: Cloud Technology & Services (34% - Compute, Storage, DB, Network), Domain 4: Billing & Support (12%).
- **Preparation & Test Strategy**: **Map Keyword Thinking** (linking services to 1-2 core keywords, e.g. SQS = Decouple/Microservices); analyzing incorrect answers during mock tests; elimination technique and identifying trick words ("Not", "Least cost").

---

### Key Takeaways

#### DevSecOps & AI Security Mindset
- **Shift-Left Security & Full Lifecycle**: Integrating security early during architecture design (Terraform/Markdown Design Review) and code pull requests (Code Security Review).
- **Autonomous Reasoning via Agentic AI**: Leveraging Amazon Bedrock to build AI Agents capable of planning pentest tasks and executing multi-step exploit chains (IDOR -> XSS).
- **Understanding AI Security Limitations**: Recognizing that while AI Agents streamline vulnerability scanning, human expertise remains crucial for complex business logic validation and handling MFA/biometric auth blocks.

#### Observability & SLA Strategy
- **Healthy Infrastructure ≠ Healthy User Experience**: A green dashboard (CPU 18%, healthy ALB hosts) does not equal happy users; monitoring must focus on the actual Customer Journey.
- **Monitoring Pyramid Model**: Layering observability from Cloud Provider → Infrastructure → Application → Business Metrics → Customer Experience.
- **Design for Failure Philosophy**: Internalizing Dr. Werner Vogels' principle (*"Everything fails all the time, so plan for failure and nothing fails"*), establishing automated alert pipelines via CloudWatch Alarms & SNS.

#### AWS Certification Strategy
- **Map Keyword Thinking Method**: Associating every AWS service with 1-2 core use-case keywords (e.g., SQS = Decouple/Microservices, S3 = Object Storage, Artifact = Audit Reports).
- **Analyzing Incorrect Answers**: Maximizing mock test value by dissecting why wrong choices are incorrect rather than simply memorizing correct options.
- **Elimination Technique & Pitfall Identification**: Eliminating irrelevant/made-up choices to boost accuracy to 50%; carefully highlighting decisive keywords ("Not", "Least cost", "Most scalable").

---

### Applying to Work & LearnSphere Project

- **Integrate Automated Security Scanning**: Implement DevSecOps practices in the **LearnSphere** repository, incorporating automated linters and security scanners into GitHub Actions to detect vulnerabilities and secret leaks prior to PR merging.
- **Establish User-Centric Custom Metrics**: Shift from basic CPU/RAM tracking to monitoring actual user experience metrics on LearnSphere (e.g., login success rates, quiz completion rates, and API response latency).
- **Configure Automated Incident Alerting**: Set up CloudWatch Alarms integrated with Amazon SNS to trigger instant Slack/Email notifications whenever LearnSphere API error rates breach defined thresholds.
- **Build an AWS Cloud Practitioner Study Plan**: Apply Map Keyword Thinking and hands-on AWS Free Tier practice to prepare for conquering the CLF-C02 certification exam.
- **Review Architecture against AWS Well-Architected Framework**: Audit the LearnSphere cloud deployment across the 6 pillars (Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability).

---

### Event Experience & Reflection

Attending **Meetup 11/07/2026** was an exciting and technically rich experience, combining the energy of a cloud architecture competition with practical DevSecOps and monitoring insights:

#### Learning from the Cloud Architect Competition
- Opening with the Cloud Architect finals created an electrifying atmosphere. Watching finalist teams (KLKAT vs Ngu Dai Hiep) analyze 10 complex scenario questions reinforced fundamental concepts around VPC Subnets, IAM Roles, and High Availability.
- Highlighted the value of rapid architectural reasoning and teamwork under time pressure.

#### Modern DevSecOps & AI Agent Exploration
- Impressed by the live demo of AWS Security Agent on Amazon Bedrock. Watching an AI agent analyze architecture docs and execute exploit chains (IDOR -> XSS) showcased the future of automated security.
- Recognized practical boundaries such as MFA blocks where human expertise remains essential.

#### Shifting Perspectives on SLA & Monitoring
- The presentation on SLA and the Monitoring Pyramid addressed a major engineering pain point: "Green infrastructure doesn't mean happy users". The `/login` failure scenario transformed my approach to observability design.
- Remembered Dr. Werner Vogels' famous quote: *"Everything fails all the time, so plan for failure and nothing fails"*.

#### Clear Roadmap for AWS Certification
- The breakdown of the AWS Cloud Practitioner (CLF-C02) exam and Map Keyword Thinking provided a clear, confident strategy for certification success.

#### Event Photos

![The intense Cloud Architect Competition Finals stage match at Bitexco Tower (KLKAT vs Ngu Dai Hiep)](/images/events/event-2/event2_stage.jpg)

![Group photo with all community members attending the Meetup on 11/07/2026 at AWS Office](/images/events/event-2/event2_group.jpg)

![Check-in at the Meetup event on 11/07/2026 on the 26th Floor of Bitexco Tower](/images/events/event-2/event2_selfie.jpg)

> **Summary:** The meetup fueled my passion for Cloud architecture, offering immense value from the Cloud Architect finals to real-world security, monitoring, and exam strategies.
