---
title: "Blog 3"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# [AWS Security & Network] Phân Tích Mô Hình Phân Vùng Mạng & Bảo Mật Hạ Tầng Cho AI Learning Platform

Chào mọi người trong **AWS Study Group**,

Hôm nay mình xin chia sẻ bài viết phân tích chi tiết về Thiết kế hạ tầng mạng (Networking) và Chiến lược phân vùng bảo mật (Security Structure) trên AWS cho dự án **LearnSphere**.

Một hệ thống vừa phục vụ người dùng tĩnh (Frontend), vừa xử lý ứng dụng (EC2 Backend), lại vừa giao tiếp với các API bên ngoài (OpenAI, MongoDB Atlas) đòi hỏi việc phân định ranh giới bảo mật cực kỳ rõ ràng.

![Sơ đồ kiến trúc hạ tầng mạng và phân vùng bảo mật LearnSphere trên AWS](/images/LEARNSHPHERE.png)

---

### 1. Thiết Kế Mô Hình Mạng (VPC & Subnet)

Hệ thống được gói gọn trong một Virtual Private Cloud (VPC) đặt tại Region Singapore (`ap-southeast-1`):

- **Public Subnet & Internet Gateway**: EC2 Instance đóng vai trò là Application Server được đặt tại Public Subnet thuộc Availability Zone (`ap-southeast-1a`). Internet Gateway (IGW) mở đường cho phép EC2 nhận các HTTP/HTTPS request trực tiếp từ người dùng và kết nối ra môi trường Internet công cộng.
- **Tối ưu băng thông mạng**: Việc đặt toàn bộ hạ tầng tại Region Singapore giúp tối ưu đáng kể latency cho đối tượng người dùng cuối tại Việt Nam và khu vực Đông Nam Á.

---

### 2. Chiến Lược Tách Biệt Lưu Trữ (S3 Buckets Isolation)

Một điểm đáng chú ý trong sơ đồ thiết kế của nhóm là việc chia tách làm 2 S3 Buckets riêng biệt thay vì dùng chung một bucket:

- **Bucket Frontend (`learnsphere-fe-static`)**: Chỉ chứa các file tĩnh đã qua biên dịch (HTML, CSS, JS, Bundle Assets). Bucket không mở public hoàn toàn mà được cấu hình để phục vụ nội dung thông qua **Amazon CloudFront**. Điều này giúp ẩn đi gốc S3 Bucket, chống các đợt tấn công càn quét tài nguyên trực tiếp vào S3.
- **Bucket Lưu trữ Dữ liệu AI (`ai-learning-platform-vhd`)**: Lưu trữ các tệp tin bài học, dữ liệu media (video, PDF, Docx) hoặc các tài nguyên phục vụ riêng cho mô hình AI. Phân quyền nghiêm ngặt tại đây chỉ cho phép EC2 Instance Backend có thẩm quyền truy xuất (Read/Write), ngăn chặn việc truy cập trái phép từ bên ngoài.

---

### 3. Kiểm Soát Luồng Giao Tiếp Ngoại Vi (Outbound Traffic)

Server EC2 Backend không chỉ đứng yên nhận request từ User, mà còn đóng vai trò là client gửi dữ liệu đi các bên thứ ba:

- **Tương tác với OpenAI API**: Luồng dữ liệu đi qua Internet Gateway để gửi prompt và nhận phản hồi từ dịch vụ OpenAI.
- **Tương tác với MongoDB Atlas**: EC2 thiết lập kết nối an toàn out-of-AWS đến cụm database MongoDB Atlas.
- **Giải pháp bảo mật cho Outbound Traffic**: Không bao giờ lưu cứng (hardcode) các chuỗi kết nối (Connection String) hay API Key trong mã nguồn. Nhóm sử dụng **AWS Systems Manager Parameter Store**, **Secrets Manager** hoặc cấu hình Environment Variables bảo mật trên EC2 để truyền chìa khóa truy cập vào Container lúc runtime. Đồng thời, thiết lập IP Whitelisting trên MongoDB Atlas để chỉ cho phép duy nhất IP tĩnh (Elastic IP) của EC2 Instance trên AWS truy cập vào Database.

---

### 4. Nhật Ký Giám Sát & Truy Vết Sự Cố (Logging & Auditing)

Để đảm bảo an toàn vận hành, mọi hoạt động trên EC2 Instance đều được đẩy về **Amazon CloudWatch (Logs & Alarms)**:

- **System & Container Logs**: Giúp phát hiện sớm các truy cập bất thường hoặc lỗi phát sinh trong quá trình container vận hành.
- **Resource Metrics**: Theo dõi CPU, RAM, Network I/O để chủ động phát hiện các dấu hiệu bị tấn công từ chối dịch vụ (DDoS) hoặc cạn kiệt tài nguyên hệ thống.

---

### 5. Định Hướng Nâng Cấp Bảo Mật (Security Roadmap)

Trong các phiên bản phát triển tiếp theo, nhóm đang lên kế hoạch tối ưu thêm cho lớp Security:

- **Đưa EC2 vào Private Subnet**: Kết hợp Application Load Balancer (ALB) ở Public Subnet và NAT Gateway cho Outbound Traffic để đưa EC2 hoàn toàn vào vùng mạng kín (Private).
- **Tích hợp AWS WAF (Web Application Firewall)**: Đặt AWS WAF trước CloudFront và ALB để chặn các lỗ hổng phổ biến như SQL Injection, Cross-Site Scripting (XSS) hay Botnet attack.

---

Mọi người có góp ý gì về cách phân vùng S3 bucket cũng như thiết kế Security Group cho EC2 trong sơ đồ này không? Rất mong nhận được thảo luận từ mọi người trong group!
