---
title: "Event 2"
date: 2026-07-11
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Bài thu hoạch “Meetup 11/07/2026”

### Mục Đích Của Sự Kiện

- **Tham gia & Cổ vũ Chung kết Cuộc thi Cloud Architect**: Theo dõi màn tranh tài kịch tính giữa 2 đội thi xuất sắc nhất qua 10 câu hỏi thử thách kiến thức chuyên sâu về đám mây AWS.
- **Khám phá ứng dụng AI trong Bảo mật Web Apps (AWS Security Agent)**: Tìm hiểu cách sử dụng Agentic AI trên nền tảng Amazon Bedrock để tự động hóa Pentest, Code Security Review và Design Review.
- **Thấu hiểu bản chất của SLA & Monitoring**: Tiếp cận mô hình Monitoring Pyramid, phân biệt giữa "Hạ tầng khỏe (Healthy Infrastructure)" và "Trải nghiệm người dùng tốt (Healthy User Experience)".
- **Chinh phục chứng chỉ AWS Certified Cloud Practitioner (CLF-C02)**: Nắm vững cấu trúc đề thi, trọng số 4 domains, tư duy Keyword Mapping và chiến lược ôn luyện thi chứng chỉ AWS hiệu quả.

---

### Danh Sách Hoạt Động & Diễn Giả

- **Hoạt động mở đầu:** *Chung kết Cuộc thi Cloud Architect* (Màn đối đầu kịch tính giữa 2 đội thi KLKAT vs Ngũ Đại Hiệp qua 10 câu hỏi kiến thức AWS Cloud).
- **Phiên 1:** **Thịnh Nguyễn** *(DevOps/DevSecOps/Cloud Engineer @ Styl Solutions, FCAJ)*
  - **Chủ đề:** *Securing Your Web Apps With AWS Security Agent*
- **Phiên 2:** **Nguyễn Huỳnh Sơn** *(Infrastructure Support Engineer @ Endava, Ex-SPS, AWS Student Builder Group HUFLIT)*
  - **Chủ đề:** *SLA and Monitoring - From SLA to Monitoring what really matters*
- **Phiên 3:** **Ngô Lê Tấn Huy** *(Cloud Practitioner Presenter)*
  - **Chủ đề:** *Inside The Exam: AWS Cloud Practitioner (CLF-C02)*

---

### Nội Dung Nổi Bật

#### Hoạt động mở đầu: Chung kết Cuộc thi Cloud Architect
- **Hình thức thi đấu**: 2 đội thi xuất sắc nhất (KLKAT vs Ngũ Đại Hiệp) cùng bước vào vòng chung kết trực tiếp trên sân khấu, tranh tài giải quyết **10 câu hỏi trắc nghiệm & tình huống thực tế về hạ tầng đám mây AWS** (VPC, IAM, EC2, S3, High Availability, Security).
- **Kết quả & Không khí**: Trận đấu diễn ra vô cùng sôi nổi với những màn rượt đuổi điểm số sát sao. Đội có chiến thuật trả lời nhanh, chính xác và đạt tổng điểm cao hơn đã xuất sắc giành chiến thắng chung cuộc.

#### 1. Securing Your Web Apps With AWS Security Agent (Thịnh Nguyễn - Styl Solutions)
- **Nghẽn bảo mật truyền thống**: Pentest thủ công tốn hàng tuần, chi phí đắt đỏ ($5k - $20k/đợt) và phụ thuộc nhiều vào trình độ con người.
- **AWS Security Agent (Frontier Agent)**: Tự động hóa nhờ Amazon Bedrock, bao phủ full lifecycle từ Design Review (Markdown/Terraform), Code Security Review (tích hợp GitHub/GitLab PR, auto-fix patch) đến Automated Pentesting (giả lập tấn công thực tế IDOR -> XSS, xuất attack graph).
- **Chi phí & Giới hạn**: Chi phí $5/Task-Hour (tiết kiệm đáng kể so với team Pentest truyền thống); tuy nhiên bị chặn bởi MFA/Biometrics và chưa tự phát hiện được các lỗi logic nghiệp vụ phức tạp.

#### 2. SLA and Monitoring - From SLA to Monitoring What Really Matters (Nguyễn Huỳnh Sơn - Endava)
- **Khái niệm SLA & Quản trị rủi ro**: SLA (Service Level Agreement) quy định cam kết chất lượng dịch vụ. Giám sát (Monitoring) nằm trong quy trình quản trị rủi ro (Identify -> Monitor -> Respond -> Improve).
- **Thông điệp cốt lõi**: **"Healthy Infrastructure ≠ Healthy User Experience"**. Giám sát hạ tầng (CPU, Memory, Disk) green 100% không đồng nghĩa người dùng happy. Case study: `/api` trả về `200 OK` nhưng `/login` thất bại do lỗi kết nối RDS DB làm trải nghiệm người dùng tụt về 0%.
- **Monitoring Pyramid & Alerting**: Xây dựng tháp giám sát từ Cloud Provider → Infrastructure → Application → Business → Customer Experience. Thiết lập alert flow tự động từ Custom Metric → CloudWatch Alarm → SNS Topic → Email/Slack.

#### 3. Inside The Exam: AWS Cloud Practitioner CLF-C02 (Ngô Lê Tấn Huy)
- **Tổng quan kỳ thi**: Bài thi nền tảng gồm 65 câu hỏi (90 phút, thang điểm 100-1000, điểm đạt 700, giá trị 3 năm).
- **Cấu trúc 4 Domains**: Domain 1: Cloud Concepts (24%), Domain 2: Security & Compliance (30% - Shared Responsibility Model, IAM), Domain 3: Cloud Technology & Services (34% - Compute, Storage, DB, Network), Domain 4: Billing & Support (12%).
- **Bí kíp ôn luyện & Làm bài**: Phương pháp **Map Keyword Thinking** (gắn service với 1-2 từ khóa cốt lõi, ví dụ SQS = Decouple/Microservices); Phân tích sâu đáp án sai khi làm mock test; Kỹ thuật loại trừ (Elimination) và đọc kỹ từ khóa bẫy ("Not", "Least cost").

---

### Những Gì Học Được

#### Tư Duy Bảo Mật & AI Agent (DevSecOps & AI Mindset)
- **Shift-Left Security & Full Lifecycle**: Tích hợp bảo mật sớm ngay từ khâu thiết kế kiến trúc (Design Review với Terraform/Markdown) và kiểm thử mã nguồn (Code Review trên Pull Request).
- **Autonomous Reasoning với Agentic AI**: Tận dụng Amazon Bedrock để xây dựng AI Agents có khả năng lập kế hoạch pentest và chủ động giả lập các chuỗi tấn công thực tế (IDOR -> XSS).
- **Thấu hiểu rào cản của AI Security**: AI Agent hỗ trợ đắc lực trong việc phát hiện lỗ hổng phổ biến nhưng vẫn bị giới hạn bởi bẫy MFA/Biometrics và chưa thay thế được con người trong kiểm thử logic nghiệp vụ phức tạp.

#### Tư Duy Giám Sát & Vận Hành Hạ Tầng (Observability & SLA Strategy)
- **Healthy Infrastructure ≠ Healthy User Experience**: Bảng điều khiển màu xanh (CPU 18%, ALB healthy) không đảm bảo người dùng hài lòng; giám sát phải bắt đầu từ trải nghiệm người dùng thực tế (Customer Journey).
- **Mô hình Monitoring Pyramid**: Xây dựng hệ thống giám sát phân tầng từ Cloud Provider → Infrastructure → Application → Business Metrics → Customer Experience.
- **Triết lý Thiết kế Chống Lỗi (Design for Failure)**: Nắm vững tư duy của Dr. Werner Vogels: *"Everything fails all the time, so plan for failure and nothing fails"*, thiết lập luồng cảnh báo tự động qua CloudWatch Alarms & SNS.

#### Chiến Lược Chinh Phục Chứng Chỉ AWS (Certification Strategy)
- **Phương pháp Map Keyword Thinking**: Khi học từng dịch vụ AWS, luôn gắn liền với 1-2 từ khóa use-case cốt lõi (ví dụ SQS = Decouple/Microservices, S3 = Object Storage, Artifact = Audit Reports).
- **Kỹ thuật Phân Tích Đáp Án Sai**: Ôn thi hiệu quả bằng cách mổ xẻ nguyên nhân tại sao các phương án còn lại sai thay vì chỉ học thuộc đáp án đúng.
- **Kỹ thuật Loại Trừ & Nhận Biết Bẫy**: Loại bỏ 2 đáp án phi lý/th bịa đặt để tăng tỷ lệ đúng lên 50%; chú ý các từ khóa bẫy quan trọng như "Not", "Least cost", "Most scalable".

---

### Ứng Dụng Vào Công Việc & Dự Án Learnsphere

- **Tích hợp Tự Động Quét Lỗ Hổng Bảo Mật**: Áp dụng quy trình DevSecOps vào repository **LearnSphere**, tích hợp các linter và security scanner vào GitHub Actions để kiểm tra code và phát hiện rò rỉ secrets trước khi merge.
- **Xây dựng Custom CloudWatch Metrics cho LearnSphere**: Thiết lập giám sát theo hướng Customer Experience — theo dõi tỷ lệ đăng nhập thành công, tỷ lệ nộp quiz hoàn tất và thời gian phản hồi API thay vì chỉ xem RAM/CPU.
- **Thiết Lập Hệ Thống Cảnh Báo Tự Động**: Cấu hình CloudWatch Alarms gửi thông báo tức thì qua Email/Slack via Amazon SNS khi tỷ lệ lỗi API của LearnSphere vượt ngưỡng cho phép.
- **Lập Lộ Trình Ôn Thi Chứng Chỉ AWS Cloud Practitioner**: Áp dụng phương pháp Map Keyword Thinking và luyện tập trên môi trường AWS Free Tier để chuẩn bị cho kỳ thi lấy chứng chỉ CLF-C02.
- **Rà Soát Kiến Trúc Theo AWS Well-Architected Framework**: Đánh giá lại hạ tầng đám mây của dự án LearnSphere theo 6 trụ cột (Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability).

---

### Trải Nghiệm Trong Event

Tham gia buổi **Meetup 11/07/2026** là một trải nghiệm vừa sôi động vừa giàu giá trị chuyên môn, kết hợp giữa tinh thần thi đấu thể thao điện tử công nghệ và kiến thức kỹ thuật Cloud/DevSecOps thực chiến. Một số trải nghiệm nổi bật:

#### Học hỏi từ cuộc thi thực chiến Cloud Architect
- Mở đầu sự kiện bằng trận chung kết kịch tính giữa 2 đội thi Cloud Architect (KLKAT vs Ngũ Đại Hiệp). Việc theo dõi các đội thi phân tích và nhanh tay trả lời 10 câu hỏi tình huống phức tạp giúp tôi củng cố lại rất nhiều kiến thức nền tảng về VPC Subnets, IAM Roles và High Availability.
- Nhận ra tầm quan trọng của phản xạ tư duy kiến trúc và kỹ năng làm việc nhóm dưới áp lực thời gian.

#### Trải nghiệm công nghệ DevSecOps & AI Agent hiện đại
- Ấn tượng mạnh mẽ với demo AWS Security Agent chạy trên nền Amazon Bedrock. Việc công cụ AI có thể tự động đọc file Markdown/Terraform để review kiến trúc và thực hiện exploit thử nghiệm (IDOR -> XSS) mở ra góc nhìn hoàn toàn mới về tương lai của kiểm thử bảo mật.
- Hiểu được các rào cản thực tế của AI Security như bẫy MFA hay các lỗi logic nghiệp vụ mà con người vẫn đóng vai trò quyết định.

#### Thay đổi góc nhìn về SLA & Monitoring
- Bài chia sẻ về SLA và Monitoring Pyramid thực sự chạm đúng nỗi đau của người làm hạ tầng: "Hệ thống green không có nghĩa người dùng vui". Ví dụ minh họa về API trả về HTTP 200 nhưng trang Login sập do kết nối Database đã giúp tôi thay đổi hoàn toàn tư duy thiết kế hệ thống giám sát.
- Học được câu nói nổi tiếng của Dr. Werner Vogels: *"Everything fails all the time, so plan for failure and nothing fails"*.

#### Định hướng con đường chinh phục chứng chỉ AWS
- Phần hướng dẫn chi tiết về cấu trúc đề thi AWS Cloud Practitioner (CLF-C02) cùng bíp kíp "Map Keyword Thinking" đã tháo gỡ tâm lý e ngại thi chứng chỉ. Tôi hiểu rằng việc làm bài test không chỉ để lấy bằng mà là quá trình hệ thống hóa bức tranh toàn cảnh về Cloud AWS.

#### Một số hình ảnh khi tham gia sự kiện

![Trận Chung kết Cuộc thi Cloud Architect đầy kịch tính trên sân khấu Bitexco (KLKAT vs Ngũ Đại Hiệp)](/images/events/event-2/event2_stage.jpg)

![Chụp ảnh kỷ niệm cùng toàn thể cộng đồng tham dự Meetup 11/07/2026 tại văn phòng AWS](/images/events/event-2/event2_group.jpg)

![Check-in tham gia buổi Meetup 11/07/2026 tại tầng 26 tòa nhà Bitexco](/images/events/event-2/event2_selfie.jpg)

> **Tổng kết:** Sự kiện đã tiếp thêm ngọn lửa đam mê công nghệ, từ tinh thần thi đấu nhiệt huyết của cuộc thi Cloud Architect đến những kiến thức bảo mật, monitoring và thi chứng chỉ vô cùng thực tế.
