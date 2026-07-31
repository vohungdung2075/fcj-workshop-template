---
title: "Các bài blogs đã đăng"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Dưới đây là danh sách các bài blog chia sẻ kiến thức chuyên môn đã đăng trên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj):

### [Blog 1 - Xây Dựng RAG Phức Tạp: Khi Nào Nên Tự Làm, Khi Nào Dùng Amazon Bedrock?](3.1-Blog1/)
Bài viết phân tích và so sánh dưới góc nhìn nghiên cứu kiến trúc giữa giải pháp Fully Managed (Knowledge Bases for Amazon Bedrock) và phương án tự xây dựng pipeline RAG tùy chỉnh. Bài viết đi sâu làm rõ các khía cạnh đánh đổi (trade-offs) về tốc độ triển khai, khả năng tùy biến trích xuất OCR/Chunking, phân quyền Multi-tenancy và tối ưu chi phí vận hành.

---

### [Blog 2 - [AWS Architecture] Phân Tích Kiến Trúc Hệ Thống Platform Học Tập Tích Hợp AI (LearnSphere)](3.2-Blog2/)
Bài viết phân tích chi tiết sơ đồ kiến trúc hệ thống backend & frontend cho dự án LearnSphere. Kiến trúc kết hợp giữa Serverless Static Hosting (S3 + CloudFront), Containerized Backend (EC2 + ECR), pipeline CI/CD tự động bằng GitHub Actions, giám sát với CloudWatch và tích hợp các dịch vụ ngoài (OpenAI API, MongoDB Atlas).

---

### [Blog 3 - [AWS Security & Network] Phân Tích Mô Hình Phân Vùng Mạng & Bảo Mật Hạ Tầng Cho AI Learning Platform](3.3-Blog3/)
Bài viết đi sâu phân tích thiết kế hạ tầng mạng (VPC, Public Subnet, IGW ở Region Singapore), chiến lược tách biệt lưu trữ 2 S3 Buckets (Frontend tĩnh & Dữ liệu AI), kiểm soát luồng giao tiếp Outbound Traffic bảo mật (Parameter Store, Secrets Manager, IP Whitelisting MongoDB Atlas), hệ thống giám sát CloudWatch và lộ trình nâng cấp bảo mật với ALB/WAF.