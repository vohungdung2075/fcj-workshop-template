---
title: "Event 3"
date: 2026-07-25
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Event Summary Report: “FCAJ - Agentic AI Build Week”

### Event Objectives

- **Attend Hackathon Showcase & Project Presentations**: Listen to real-world development experiences and live project demos from 4 outstanding teams at the Agentic AI Build Week.
- **Explore Practical AI Agent Architectures on AWS**: Discover enterprise architectures combining Amazon Bedrock, AgentCore Runtime, Strands Agent, SageMaker, and MCP (Model Context Protocol).
- **Learn End-to-End AI Product Engineering**: Internalize the 24-hour hackathon journey from ideation and technical troubleshooting to operational cost optimization.
- **Apply Agentic AI Concepts to Personal Projects**: Synthesize lessons on Multi-Agent Orchestration and intelligent AI Copilot design for the LearnSphere platform.

---

### Teams & Hackathon Projects

1. **Signal Scout Team** *(Members: Le Tan Luc, Do Hoang Hieu, Trieu Quoc Hao, Nguyen Van Duy Khiem, Nguyen Cong Minh, Nguyen Tran Minh Quan)*
   - **Project:** *Signal Scout - Corporate Strategic Change Signals Detection Platform*
2. **Dream AI Team** *(Project S.H.E.P.H.E.R.D)*
   - **Project:** *S.H.E.P.H.E.R.D - Smart Human-flow Evaluation, Prediction, Hazard Detection, Response, and Dispatch*
3. **Plan V Team** *(Members: Pham Tien Thuan Phat, Huynh Hoang Long, Le Minh Nghia, Tran Dai Vi, Nguyen An)*
   - **Project:** *Solution Architect Professional Native App - Automated AWS Architecture Design Assistant*
4. **OneTeam (AWS Track Winner)** *(Members: Anh Duy, Tran Dong, Doan Trung, Minh Viet, Anshul Roy)*
   - **Project:** *KFC Bot Agent - Multi-Channel Conversational Ordering AI Agent*

---

### Key Highlights

#### 1. Signal Scout - Automated Corporate Change Detection Platform
- **Problem & Solution**: Automatically ingests, validates, and analyzes public corporate restructuring signals to assist corporate strategy and enterprise risk management teams.
- **Multi-Agent Architecture**: Built with API Gateway, AWS Lambda, and **AgentCore Runtime Management**. Separated into a **Crawler Subagent** (combining TinyFish/Apify crawlers) and an **Analysis Subagent** (utilizing Bedrock Guardrails, Strands Agent, and Langfuse tracing).
- **Cost Optimization & MCP Protocol**: Designed for extreme cost efficiency (~$17 - $130/month for AWS infra) by integrating **AgentCore Gateway with Model Context Protocol (MCP)** for WebSearch and Browser tool invocation.

#### 2. S.H.E.P.H.E.R.D - Crowd Density Monitoring & Smart Dispatch (Dream AI Team)
- **Problem & Solution**: Replaces slow manual crowd monitoring at large venues; analyzes live camera streams to measure crowd density, predict queue congestion, and trigger real-time dispatch alerts.
- **Computer Vision & Cloud AI Integration**: Combined **YOLO + ByteTrack** algorithms processing video streams from Kinesis Video Streams on AWS ECS Clusters, routing data to **Amazon SageMaker Endpoints** for cloud inference.
- **Agentic AI Layer**: Developed an **Autonomous Monitor** (continuous tracking) and an **Operator Copilot** (natural language querying for staff) powered by Amazon Bedrock AgentCore & Strands Agent alongside a React Dashboard.

#### 3. Solution Architect Professional Native App (Plan V Team)
- **Problem**: Solution Architects (SAs) spend days manually reading BRD/PRD documents line by line, drafting architecture diagrams, and estimating cloud costs from scratch.
- **AI-Native App Solution**: Parses natural language project requirements, drafts high-level architecture options, generates editable Drawio diagrams with official AWS Architecture Icons, and calculates directional AWS cost estimates for `ap-southeast-1`.
- **Cloud Architecture**: Deployed on ECS Fargate inside VPC Private Subnets, backed by PostgreSQL, S3 Artifacts, Amazon EFS, Cognito authentication, and custom **Draw.io MCP** & **AWS Pricing MCP** integrations.

#### 4. KFC Bot Agent - Multi-Channel Conversational Ordering (OneTeam - AWS Track Winner)
- **Problem & Breakthrough**: Conversational ordering fails with naive chatbots due to messy natural language and strict business rules. KFC Bot Agent enables direct ordering inside Zalo OA / WhatsApp without downloading an app or switching channels.
- **5-Step Agentic Workflow**: `Goal` (understand intent) → `Plan` (determine steps) → `Tools` (search menu/promotions) → `Act` (update cart) → `Verify` (confirm against real cart state).
- **Architecture & Record Performance**: Built on WAF, API Gateway, SQS, Bedrock Agentcore, DynamoDB (Session/State), and OpenSearch (Vector Store). Achieved **3-5s end-to-end latency**, **$0.006 per order cost**, and **-60% infra code reduction** using AgentCore.

---

### Key Takeaways

#### Multi-Agent & Tooling Mindset
- **Multi-Agent Orchestration**: Understood decomposing complex workflows across specialized Subagents (Crawler Agent, Analysis Agent, Monitor Agent) rather than relying on a single monolithic LLM.
- **Model Context Protocol (MCP)**: Discovered the power of MCP in standardizing connections between AI Agents and external tools (WebSearch, Browser tool, Drawio, AWS Pricing API).
- **Design Once, Deploy Everywhere**: Implemented modular architectures that allow seamless expansion to new messaging channels (Zalo, WhatsApp, Messenger) without rebuilding core business logic.

#### Cloud Optimization & Latency Control
- **Cost Efficiency in Practice**: Learned techniques to control LLM token expenses (Bedrock accounting for ~75% of costs) and keep total cloud infrastructure under $88/month or $0.006 per transaction.
- **Real-Time Processing**: Combined Kinesis Video Streams, SageMaker, and WebSockets/Lambda to maintain end-to-end response latency under 3-5 seconds.

#### Hackathon Experience & Execution Mindset
- **Small, Finished Work Beats Big, Broken Ideas**: Completing a streamlined, working MVP is far more valuable than presenting unfinished large-scale concepts.
- **Team Synergy & Pressure Management**: Learned how complementary skills (AI, Software Engineering, Infrastructure) unite to overcome 24-hour high-pressure challenges.

---

### Applying to Work & LearnSphere Project

- **Integrate Agentic AI Copilot into LearnSphere**: Apply the **Operator Copilot** pattern and the 5-step Agentic Workflow (`Goal → Plan → Tools → Act → Verify`) to build an AI assistant for **LearnSphere** students, delivering instant Q&A and personalized lesson recommendations.
- **Leverage MCP & Vector Stores**: Explore AgentCore Gateway and OpenSearch Vector Stores for fast, accurate course document and video transcript retrieval.
- **Optimize Infrastructure & Cloud Costs**: Apply cost management practices from the Hackathon winning team to LearnSphere, using SQS, Lambda, and DynamoDB for asynchronous workloads to minimize operational expenses.
- **Embrace "Small & Finished" Philosophy**: Focus on polishing core platform features iteratively, ensuring high stability and deployment-ready quality.

---

### Event Experience & Reflection

Attending the **FCAJ - Agentic AI Build Week Hackathon Showcase** was an exhilarating experience, offering a front-row view of how young engineers transform bold AI ideas into functional products:

#### Live Demos & Presentation Impressions
- Witnessed confident live demos from 4 teams: Signal Scout's strategic signal detection, S.H.E.P.H.E.R.D's crowd monitoring, SA Native App's automated cloud design, and KFC Bot Agent's **AWS Track Winner** conversational ordering solution.
- Appreciated how teams tackled natural language edge cases and real-world system failures.

#### Vibrant Hackathon Energy
- Enjoyed authentic stories of the 24-hour rush: "coding through the night, drinking Red Bull, eating KFC, debugging at 3 AM". Stories of conquering initial doubts (*"Fear of failing, not skilled enough"*) provided immense inspiration.
- Experienced the camaraderie and mutual support among teammates and community mentors.

#### Expanded Horizons & Tech Leverage
- Realized AI is no longer just theoretical—it is a practical engineering tool tightly integrated with AWS infrastructure (Bedrock, AgentCore, SageMaker, Lambda).
- Internalized the invaluable lesson: *"Showing up is already half the battle"* and recognized the lasting power of community connections.

#### Event Photos

![Check-in photo with friend at the FCAJ Agentic AI Build Week event](/images/events/event-3/event3_selfie.jpg)

![Audience watching the Hackathon Showcase presentations in the hall](/images/events/event-3/event3_audience.jpg)

> **Summary:** The Build Week Hackathon Showcase provided deep, practical insights into Agentic AI, inspiring me to integrate cutting-edge AI technologies into the LearnSphere project and advance my Cloud AI career.
