---
title: "Event 1"
date: 2026-06-13
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Bài thu hoạch “Meetup 13/06/2026”

### Mục Đích Của Sự Kiện

- **Khám phá thiết kế kiến trúc Cloud trên AWS**: Học cách xây dựng hệ thống URL shortening có độ sẵn sàng cao, mở rộng linh hoạt và tối ưu độ trễ.
- **Thấu hiểu văn hóa và môi trường làm việc tại MNCs**: Trang bị tư duy phản biện, kỹ năng kể chuyện dữ liệu (Data Storytelling) và lộ trình phát triển nghề nghiệp.
- **Giải mã công việc thực tế của DevOps Engineer**: Xóa bỏ các ngộ nhận về DevOps, định hình lộ trình học tập và tư duy hệ thống thực chiến.
- **Định hướng phát triển từ Sinh viên đến Chuyên gia AWS**: Nắm bắt lộ trình 8 bước phát triển từ sinh viên FCAJ đến chuyên gia tại các doanh nghiệp đối tác AWS Partner.

---

### Danh Sách Diễn Giả & Các Bài Thuyết Trình

1. **Đinh Trung Kiên** *(Lead Developer at Startup)* & **Nguyễn Minh Thọ** *(Student)*
   - **Chủ đề:** *A scalable URL shortening service on AWS*
2. **Mr. Đạt Phạm** *(Data Analytics Engineer)* & **Mr. Cường Nguyễn** *(Process Engineer)*
   - **Chủ đề:** *Câu chuyện thực tế đến văn hóa tại tập đoàn đa quốc gia*
3. **Trương Hữu Trọng** *(DevOps Engineer @ Endava Vietnam)*
   - **Chủ đề:** *What does a DevOps Engineer really do?*
4. **Danh Hoàng Hiếu Nghị** *(AI Engineer – AWS Community Builder – AWS Student Builder Group Leader)*
   - **Chủ đề:** *From First Cloud AI Journey to AWS Partner*

---

### Nội Dung Nổi Bật

#### 1. A Scalable URL Shortening Service on AWS (Đinh Trung Kiên & Nguyễn Minh Thọ)
- **Đặt vấn đề**: Luồng truyền thống (User → FE → BE → DB) dễ triển khai nhưng gặp rủi ro bảo mật, độ trễ đọc cao, điểm lỗi đơn lẻ (SPOF) và rất khó mở rộng khi tải tăng cao.
- **Kiến trúc Cloud AWS**: Sử dụng Route 53, CloudFront CDN kết hợp WAF ở lớp Edge; Backend triển khai containerized trên Multi-AZ **AWS Fargate (ECS)** phía sau Application Load Balancer (ALB).
- **Key Generation Service (KGS)**: Chạy một service ngầm trên ECS để pre-generate trước short code ngẫu nhiên và đẩy vào Redis queue (`LPUSH/RPOP`), giúp khởi tạo URL tức thì không lo trùng lặp.
- **Cache-aside Pattern**: Kết hợp **Amazon DynamoDB** và **Amazon ElastiCache for Redis**; ưu tiên đọc từ Redis (*Cache Hit*), chỉ query DynamoDB khi *Cache Miss* để giảm tải tối đa cho DB chính.

#### 2. Câu Chuyện Thực Tế Đến Văn Hóa Tại Tập Đoàn Đa Quốc Gia (Mr. Đạt Phạm & Mr. Cường Nguyễn)
- **Công việc Data Analytics**: So sánh phân tích vận hành GMV tại Kamereo và tối ưu dữ liệu thiết bị IoT nhà máy tại Colgate-Palmolive.
- **Bộ 4 kỹ năng cốt lõi**: Tư duy phản biện (*Psychology*), Kỹ năng giao tiếp (*Forum*), Kể chuyện với dữ liệu (*Data Storytelling*) và Giải quyết vấn đề (*Problem Solving*).
- **Lộ trình phát triển**: Mô hình 5 cấp độ sự nghiệp (`Follower` → `Learner` → `Problem Solver` → `System Thinker` → `Super Star`).
- **Văn hóa MNCs**: Quy trình tuyển dụng 4 vòng (ATS, Test, STAR interview, Cultural fit) và tinh thần văn hóa *No-Blame Post-Mortem* (tìm nguyên nhân gốc rễ thay vì đổ lỗi cá nhân).

#### 3. What Does a DevOps Engineer Really Do? (Trương Hữu Trọng - Endava Vietnam)
- **Giải mã vai trò DevOps**: Xóa bỏ ngộ nhận "chỉ làm CI/CD hay trực đêm fix bug"; khẳng định phạm vi công việc phụ thuộc vào quy mô dự án, độ phức tạp sản phẩm và độ trưởng thành hạ tầng.
- **Công việc thực chiến**: Giải quyết các bài toán sự cố môi trường, hóa đơn cloud tăng vọt, bảo mật và đóng vai trò kết nối giữa Dev và Ops (On-call, Incident postmortem, Cost investigation).
- **Lộ trình học tập**: Nắm chắc nền tảng (Linux, Networking, Python/Golang, Git, Containers) → Hiểu cách ứng dụng vận hành → Thực hành làm các project nhỏ (break & fix).
- **Nguyên tắc làm nghề**: Hỏi "Tại sao" trước khi hỏi "Làm thế nào", tư duy hệ thống (*System Thinking*) thay vì chỉ sửa lỗi vặt, tận dụng AI để nâng cao năng suất.

#### 4. From First Cloud AI Journey to AWS Partner (Danh Hoàng Hiếu Nghị)
- **Lộ trình trưởng thành**: Từ sự tò mò ban đầu (`Student Curiosity`) → tham gia cộng đồng (`First Cloud Journey`) → thực hành project (`Portfolio`) → trở thành chuyên gia (`AWS Partner`) → chia sẻ lại (`Share Back`).
- **Định hướng sự nghiệp**: Chia sẻ kinh nghiệm phát triển qua các vị trí Solutions Architect, DevOps, AI Engineer và cơ hội việc làm tại các đối tác hàng đầu như Renova Cloud (*AWS Partner of the Year 2026*).

---

### Những Gì Học Được

#### Tư Duy Thiết Kế & Hệ Thống (System Design Mindset)
- **Separation of Concerns**: Tách biệt hoàn toàn luồng đọc (read) và luồng ghi (write) để tối ưu hóa hiệu năng theo từng loại traffic pattern thay vì chia sẻ một nút thắt cổ chai đơn lẻ.
- **Pre-computation over On-demand**: Tạo trước dữ liệu (pre-generate short code bằng KGS) giúp xử lý yêu cầu tức thì, tránh tính toán lại dưới áp lực truy cập cao.
- **System Thinking vs. Fix vặt**: Chuyển đổi tư duy từ người làm theo checklist (*Follower*) sang người nhìn nhận bài toán ở bức tranh toàn cảnh (*System Thinker*), đánh giá tác động hệ thống lâu dài thay vì sửa lỗi tạm thời.

#### Kiến Trúc Kỹ Thuật Cloud & Caching (Technical Architecture)
- **Cache-aside Pattern**: Sử dụng bộ nhớ trong Redis để phục vụ truy vấn trước; chỉ tra cứu cơ sở dữ liệu DynamoDB khi xảy ra *Cache Miss*, giữ độ trễ cực thấp và giảm tải tối đa cho DB chính.
- **Defense at the Edge**: Đẩy bảo mật (WAF) và caching (CloudFront CDN) ra gần người dùng nhất có thể, ngăn chặn các mối đe dọa và lưu lượng bất thường trước khi chạm vào core system.
- **Compute Spectrum**: Lựa chọn giải pháp compute phù hợp (AWS ECS Fargate Multi-AZ) để vừa giảm công sức quản lý server vừa bảo đảm khả năng tự động mở rộng (Auto-scaling).

#### Định Hướng Sự Nghiệp & Văn Hóa Doanh Nghiệp (Career & Corporate Strategy)
- **Data Storytelling**: Kỹ năng chuyển hóa các con số khô khan (GMV, Fulfillment, Fill Rate) thành các câu chuyện giá trị có ý nghĩa thúc đẩy hành động kinh doanh.
- **Văn hóa No-Blame Post-Mortem**: Khi xảy ra lỗi hệ thống nghiêm trọng, tập trung tìm nguyên nhân gốc rễ để cải tiến hệ thống thay vì đổ lỗi cho cá nhân.
- **Lộ trình phát triển**: Từ tò mò sinh viên (`Student Curiosity`) → thực hành project (`Portfolio`) → trở thành chuyên gia tại các đối tác đám mây (`AWS Partner`).

---

### Ứng Dụng Vào Công Việc & Dự Án Learnsphere

- **Áp dụng Cache-aside & Presigned URLs**: Tích hợp ElastiCache Redis và S3 Presigned URLs vào dự án **LearnSphere** để tối ưu hóa tốc độ xem video bài học và tải tài liệu PDF, giảm tối đa tải cho cơ sở dữ liệu MongoDB.
- **Chuẩn hóa Containerization & CI/CD**: Áp dụng tư duy DevOps thực chiến để viết Dockerfile tối ưu và xây dựng GitHub Actions deployment pipeline tự động cho backend Node.js/Express.
- **Tối ưu hóa quy trình Log & Monitoring**: Thiết lập giám sát CloudWatch Logs và cảnh báo tự động cho các API nòng cốt (đăng ký, đăng nhập, nộp bài quiz) thay vì chỉ theo dõi CPU server.
- **Xây dựng tư duy Data Storytelling trong báo cáo**: Áp dụng cách biểu diễn sơ đồ kiến trúc hệ thống và số liệu vận hành bài bản vào bài báo cáo thực tập FCJ.
- **Thực hành văn hóa Code Review & No-Blame**: Xây dựng quy trình kiểm thử và review code chặt chẽ trong nhóm phát triển, tập trung vào sửa đổi hệ thống và cải thiện chất lượng sản phẩm.

---

### Trải Nghiệm Trong Event

Tham gia buổi **Meetup 13/06/2026** là một hành trình học hỏi vô cùng thực tế và truyền cảm hứng, mang lại góc nhìn đa chiều từ kỹ thuật chuyên sâu đến kỹ năng định hướng sự nghiệp. Một số trải nghiệm nổi bật:

#### Học hỏi từ các diễn giả thực chiến
- Được trực tiếp lắng nghe những chia sẻ rất thật về công việc hàng ngày của một Data Analytics Engineer và DevOps Engineer tại các tập đoàn lớn (Colgate-Palmolive, Endava Vietnam). Những bài học từ thực tế đã giúp tôi giải đáp được nhiều thắc mắc về khoảng cách giữa kiến trúc trên lý thuyết và bài toán vận hành ngoài đời thực.
- Ấn tượng mạnh mẽ với mô hình 5 cấp độ phát triển bản thân (từ *Follower* nâng cấp dần lên *Problem Solver* và *System Thinker*), giúp tôi xác định rõ cột mốc cần hướng tới trong sự nghiệp.

#### Trải nghiệm nội dung chuyên môn phong phú
- Các phiên thuyết trình đi sâu vào những case study sinh động: từ cách giải quyết bài toán rút ngắn URL chịu tải cao bằng Key Generation Service & Redis Cache, cho đến các bức tranh toàn cảnh về chuỗi cung ứng và dữ liệu sản xuất nhà máy.
- Cách diễn giả minh họa về vai trò "thầm lặng nhưng nòng cốt" của vị trí DevOps cũng như bức tranh tổng quan về hệ sinh thái công cụ Cloud hiện đại giúp tôi định hình rõ ràng hơn con đường kỹ sư hạ tầng.

#### Kết nối cộng đồng & Mở rộng định hướng
- Hiểu rõ hơn về hệ sinh thái và các hoạt động của cộng đồng sinh viên công nghệ như **AWS Student Builder Group** và chương trình **First Cloud AI Journey (FCAJ)**. Hành trình thực tế từ sinh viên đến khi làm việc tại đối tác **AWS Partner** của diễn giả đã tạo động lực rất lớn, giúp tôi chủ động hơn trong việc tìm kiếm cơ hội học tập, làm project thực chiến và networking.

#### Bài học rút ra
- Luôn giữ tinh thần tò mò, tư duy chủ động học hỏi (*Learner mindset*) và thói quen đặt câu hỏi "Tại sao" trước khi bắt tay vào triển khai bất kỳ công nghệ nào.
- Chuẩn bị một hành trang phát triển toàn diện: kết hợp giữa nền tảng kỹ thuật vững chắc (Linux, Cloud Architecture, Caching), kỹ năng mềm (Data Storytelling, giao tiếp) và sự thấu hiểu văn hóa doanh nghiệp (*No-blame culture*).

#### Một số hình ảnh khi tham gia sự kiện

![Chụp ảnh kỷ niệm cùng cộng đồng AWS Student Builders tại văn phòng AWS Việt Nam](/images/events/event-1/event1_group.jpg)

![Check-in tham gia buổi Meetup tại tầng 26 tòa nhà Bitexco](/images/events/event-1/event1_selfie.jpg)

> **Tổng kết:** Sự kiện không chỉ mang đến những lượng kiến thức Cloud & DevOps giá trị mà còn mở rộng tầm nhìn nghề nghiệp, giúp tôi tự tin hơn trên con đường phát triển chuyên môn tại AWS Bootcamp.
