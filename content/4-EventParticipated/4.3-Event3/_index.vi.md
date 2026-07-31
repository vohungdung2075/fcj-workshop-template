---
title: "Event 3"
date: 2026-07-25
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Bài thu hoạch “FCAJ - Agentic AI Build Week”

### Mục Đích Của Sự Kiện

- **Tham gia Showcase & Thuyết trình Hackathon Agentic AI**: Lắng nghe trải nghiệm thực chiến và phần thuyết trình demo dự án từ 4 đội thi xuất sắc tại tuần lễ Agentic AI Build Week.
- **Tiếp cận các ứng dụng AI Agent thực chiến trên AWS**: Khám phá kiến trúc tích hợp Amazon Bedrock, AgentCore Runtime, Strands Agent, SageMaker và MCP (Model Context Protocol).
- **Học hỏi kinh nghiệm phát triển sản phẩm AI end-to-end**: Thấu hiểu hành trình 24 giờ thử thách từ khâu lên ý tưởng, vượt qua khó khăn kỹ thuật đến việc tối ưu hóa chi phí vận hành cho ứng dụng AI.
- **Vận dụng tư duy Agentic AI vào dự án cá nhân**: Đúc kết bài học về thiết kế Multi-agent Orchestration và ứng dụng tính năng trợ lý AI thông minh cho nền tảng LearnSphere.

---

### Danh Sách Các Đội Thi & Dự Án Hackathon

1. **Đội Signal Scout** *(Thành viên: Lê Tấn Lực, Đỗ Hoàng Hiếu, Triệu Quốc Hào, Nguyễn Văn Duy Khiêm, Nguyễn Công Minh, Nguyễn Trần Minh Quân)*
   - **Dự án:** *Signal Scout - Nền tảng phát hiện tín hiệu thay đổi chiến lược doanh nghiệp*
2. **Đội Dream AI Team** *(Project S.H.E.P.H.E.R.D)*
   - **Dự án:** *S.H.E.P.H.E.R.D - Hệ thống giám sát, dự báo mật độ đám đông và điều phối thông minh*
3. **Đội Plan V** *(Thành viên: Phạm Tiến Thuận Phát, Huỳnh Hoàng Long, Lê Minh Nghĩa, Trần Đại Vĩ, Nguyễn An)*
   - **Dự án:** *Solution Architect Professional Native App - Trợ lý AI tự động hóa thiết kế kiến trúc đám mây AWS*
4. **Đội OneTeam (Đội đạt giải AWS Track Winner)** *(Thành viên: Anh Duy, Trần Đông, Đoàn Trung, Minh Việt, Anshul Roy)*
   - **Dự án:** *KFC Bot Agent - Multi-Channel AI Agent đặt hàng qua trò chuyện*

---

### Nội Dung Nổi Bật

#### 1. Signal Scout - Nền Tảng Tự Động Phân Tích Tín Hiệu Doanh Nghiệp
- **Bài toán & Giải pháp**: Tự động thu thập, kiểm chứng bằng chứng và phân tích các tín hiệu thay đổi chiến lược, tái cấu trúc của doanh nghiệp từ dữ liệu công khai, hỗ trợ các đội ngũ chiến lược và quản trị rủi ro.
- **Kiến trúc Multi-Agent**: Sử dụng API Gateway, AWS Lambda kết hợp **AgentCore Runtime Management**. Tách biệt thành **Crawler Subagent** (kết hợp TinyFish/Apify crawler) và **Analysis Subagent** (dùng Bedrock Guardrails, Strands Agent và Langfuse tracing).
- **Tối ưu chi phí & Mô hình MCP**: Thiết kế kiến trúc tối ưu chi phí (chỉ từ ~$17 - $130/tháng cho AWS infra) bằng cách tích hợp **AgentCore Gateway với giao thức MCP (Model Context Protocol)** để gọi WebSearch & Browser tools.

#### 2. S.H.E.P.H.E.R.D - Hệ Thống Giám Sát & Điều Phối Mật Độ Đám Đông (Dream AI Team)
- **Bài toán & Giải pháp**: Giải quyết vấn đề giám sát thủ công chậm chạp tại các sự kiện đông người; phân tích camera gian hàng/lối vào để đo mật độ đám đông, dự báo ùn tắc và phát cảnh báo điều phối theo thời gian thực.
- **Tích hợp Computer Vision & Cloud AI**: Kết hợp thuật toán **YOLO + ByteTrack** xử lý video stream từ Kinesis Video Streams trên AWS ECS Cluster, truyền dữ liệu qua **Amazon SageMaker Endpoint** để suy luận.
- **Lớp Agentic AI**: Xây dựng **Autonomous Monitor** (giám sát liên tục) và **Operator Copilot** (cho phép ban quản lý truy vấn ngôn ngữ tự nhiên) dựa trên Amazon Bedrock AgentCore & Strands Agent kết hợp React Dashboard.

#### 3. Solution Architect Professional Native App (Đội Plan V)
- **Bài toán**: Các Solution Architect (SA) mất nhiều ngày đọc tài liệu BRD/PRD, vẽ sơ đồ Drawio và tính toán chi phí AWS thủ công từ trang giấy trắng.
- **Giải pháp AI Native App**: Ứng dụng phân tích yêu cầu bằng ngôn ngữ tự nhiên, tự động sinh sơ đồ Drawio chuẩn AWS Architecture Icons, xuất dự toán chi phí AWS (ap-southeast-1) và gợi ý thiếu sót yêu cầu qua giao diện Chat Sidebar.
- **Kiến trúc Cloud**: Triển khai trên ECS Fargate trong VPC Private Subnet, kết hợp PostgreSQL, S3 Artifacts, Amazon EFS, Cognito authentication và tích hợp các công cụ **Draw.io MCP** & **AWS Pricing MCP**.

#### 4. KFC Bot Agent - Multi-Channel AI Agent Đặt Hàng Qua Chat (Đội OneTeam - AWS Track Winner)
- **Bài toán & Đột phá**: Đặt hàng qua chat thường thất bại nếu chỉ dùng chatbot thông thường do ngôn ngữ tự nhiên phức tạp và quy định khuyến mãi khắt khe. KFC Bot Agent cho phép người dùng đặt hàng trực tiếp trên Zalo OA / WhatsApp mà không cần tải app hay chuyển ứng dụng.
- **Quy trình 5 bước Agentic Workflow**: `Goal` (hiểu ý định) → `Plan` (lập kế hoạch) → `Tools` (tra cứu menu/khuyến mãi) → `Act` (cập nhật giỏ hàng) → `Verify` (xác nhận đơn hàng thực tế).
- **Kiến trúc & Hiệu năng kỷ lục**: Hạ tầng dựa trên WAF, API Gateway, SQS, Bedrock Agentcore, DynamoDB (Session/State) và OpenSearch (Vector Store). Đạt hiệu năng **3-5s độ trễ end-to-end**, chi phí cực rẻ **$0.006/đơn hàng** và giảm **-60% dung lượng code hạ tầng** nhờ AgentCore.

---

### Những Gì Học Được

#### Tư Duy Thiết Kế Multi-Agent & MCP (Multi-Agent & Tooling Mindset)
- **Multi-Agent Orchestration**: Thấu hiểu cách phân tách nhiệm vụ giữa các Subagents chuyên biệt (Crawler Agent, Analysis Agent, Monitor Agent) để xử lý luồng công việc phức tạp thay vì dùng một LLM đơn lẻ.
- **Giao thức MCP (Model Context Protocol)**: Khám phá sức mạnh của MCP trong việc chuẩn hóa kết nối giữa AI Agent với các công cụ bên ngoài (WebSearch, Browser tool, Drawio, AWS Pricing API).
- **Tư duy thiết kế "Design Once, Deploy Everywhere"**: Xây dựng kiến trúc modular giúp dễ dàng mở rộng thêm kênh giao tiếp mới (Zalo, WhatsApp, Messenger) hoặc tính năng mới mà không phải làm lại từ đầu.

#### Tối Ưu Chi Phí & Độ Trễ Đám Mây (Cloud Optimization & Latency)
- **Tối ưu chi phí thực chiến**: Bài học kiểm soát chi phí LLM token (Bedrock chiếm ~75% chi phí) và hạ tầng Cloud xuống mức siêu rẻ ($0.006/đơn hàng hoặc ~$88/tháng).
- **Xử lý thời gian thực (Real-time Processing)**: Kết hợp Kinesis Video Streams, SageMaker và WebSocket/Lambda để giữ độ trễ phản hồi end-to-end chỉ từ 3-5 giây.

#### Bài Học Thực Chiến Từ Hackathon (Hackathon Experience & Mindset)
- **Small, finished work beats big, broken ideas**: Việc hoàn thiện một sản phẩm nhỏ, chạy mượt mà quan trọng hơn tạo ra ý tưởng lớn nhưng dở dang.
- **Tinh thần đồng đội & Vượt qua giới hạn**: Cách phối hợp giữa các thành viên có thế mạnh khác nhau (AI, Software Engineering, Infrastructure) trong thử thách 24 giờ áp lực cao.

---

### Ứng Dụng Vào Công Việc & Dự Án Learnsphere

- **Tích hợp Agentic AI Copilot cho LearnSphere**: Áp dụng mô hình **Operator Copilot** và quy trình 5 bước Agentic Workflow (`Goal → Plan → Tools → Act → Verify`) để xây dựng trợ lý AI hỗ trợ học viên giải đáp thắc mắc và tự động đề xuất bài học trong **LearnSphere**.
- **Ứng dụng MCP & Vector Store**: Nghiên cứu sử dụng AgentCore Gateway và OpenSearch Vector Store để tra cứu tài liệu học tập và tài nguyên khóa học nhanh chóng, chính xác.
- **Tối ưu hạ tầng & Chi phí Cloud**: Áp dụng bài học kiểm soát chi phí từ đội chiến thắng Hackathon vào LearnSphere, sử dụng SQS, Lambda và DynamoDB cho các tác vụ bất đồng bộ để giữ chi phí vận hành ở mức tối thiểu.
- **Áp dụng tư duy "Small & Finished"**: Tập trung hoàn thiện các tính năng cốt lõi của ứng dụng thay vì lan man, đảm bảo sản phẩm luôn trong trạng thái ổn định và sẵn sàng demo/deploy.

---

### Trải Nghiệm Trong Event

Tham gia buổi **Showcase & Thuyết trình Hackathon FCAJ - Agentic AI Build Week** là một trải nghiệm bùng nổ cảm xúc, mang lại bức tranh toàn cảnh về cách các kỹ sư trẻ biến ý tưởng AI táo bạo thành sản phẩm thực tế:

#### Ấn tượng từ các phần thuyết trình & Demo thực chiến
- Được chứng kiến các màn Demo trực tiếp (Live Demo) đầy tự tin từ 4 đội thi: từ hệ thống phân tích tín hiệu doanh nghiệp Signal Scout, hệ thống giám sát mật độ đám đông S.H.E.P.H.E.R.D, ứng dụng trợ lý thiết kế kiến trúc SA Native App, đến giải pháp KFC Bot Agent xuất sắc giành giải **AWS Track Winner**.
- Thích thú trước cách các đội giải quyết bài toán giao tiếp tự nhiên và xử lý lỗi thực tế (như case study bài học từ thử nghiệm AI drive-thru của McDonald's).

#### Trải nghiệm tinh thần Hackathon sôi nổi
- Lắng nghe những chia sẻ rất thật về hành trình 24 giờ "thức trắng đêm, uống 5 lon Redbull, ăn KFC, debug lúc 3h sáng" của các đội thi. Những câu chuyện vượt qua nỗi sợ ban đầu (*"Fear of failing, not skilled enough"*) đã truyền cảm hứng mãnh liệt về tinh thần dám nghĩ dám làm.
- Cảm nhận không khí gắn kết, hỗ trợ lẫn nhau giữa các thành viên trong team và giữa các nhóm thi đấu cùng cộng đồng AWS.

#### Mở rộng tư duy & Đòn bẩy công nghệ
- Nhận ra rằng AI không còn là lý thuyết xa vời mà đã trở thành công cụ thực chiến gắn liền với hạ tầng AWS (Bedrock, AgentCore, SageMaker, Lambda).
- Học được bài học quý giá: *"Showing up is already half the battle"* (Có mặt và bắt tay vào làm đã là thắng lợi một nửa) và giá trị của việc kết nối cộng đồng.

#### Một số hình ảnh khi tham gia sự kiện

![Chụp ảnh kỷ niệm check-in cùng bạn tại sự kiện FCAJ Agentic AI Build Week](/images/events/event-3/event3_selfie.jpg)

![Toàn cảnh hội trường các tham dự viên theo dõi buổi Hackathon Showcase](/images/events/event-3/event3_audience.jpg)

> **Tổng kết:** Sự kiện Build Week Hackathon Showcase đã mang đến cho em cái nhìn thực tế và sâu sắc nhất về Agentic AI, tiếp thêm động lực mạnh mẽ để ứng dụng công nghệ AI tiên tiến vào dự án LearnSphere và phát triển sự nghiệp Cloud AI.
