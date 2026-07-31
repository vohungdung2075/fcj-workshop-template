---
title: "Bản đề xuất"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# LearnSphere - Online Learning Platform Integrated with AI
## Giải pháp Học tập Hiện đại trên Nền tảng Đám mây AWS

### 1. Tóm Tắt Điều Hành

**LearnSphere** là một nền tảng học tập trực tuyến tích hợp AI thế hệ mới dành cho học viên, giảng viên và quản trị viên. Hệ thống tập trung quản lý khóa học, bài học, media (video, tài liệu PDF/DOCX), quiz, tiến độ học tập và thảo luận tương tác trong cùng một ứng dụng web SPA hiện đại. Trợ lý AI Assistant hỗ trợ học viên giải đáp thắc mắc theo đúng ngữ cảnh bài học, tóm tắt tài liệu tự động và hỗ trợ giảng viên sinh câu hỏi quiz theo độ khó.

Giải pháp sử dụng kiến trúc High Availability (HA) triển khai tại AWS Region Singapore (`ap-southeast-1`). Frontend React/Vite/TypeScript được build thành các tài nguyên tĩnh, lưu trữ riêng tư trong Amazon S3 Bucket Frontend và phân phối toàn cầu qua Amazon CloudFront. Tên miền chính thức `https://www.learnspherev2.id.vn` được quản lý DNS tại TenTen, trỏ CNAME tới CloudFront và bảo mật bằng chứng chỉ HTTPS do AWS Certificate Manager (ACM) cấp.

Backend Node.js/Express được đóng gói thành Docker container bất biến, lưu trữ tại Amazon ECR và vận hành thông qua Auto Scaling Group duy trì tối thiểu 2 EC2 instances trên 2 Availability Zones (`ap-southeast-1a` và `ap-southeast-1b`) nằm trong các Private Subnet, đứng sau Application Load Balancer. Tài nguyên media khóa học được lưu trong Amazon S3 Media Bucket riêng biệt và truy xuất an toàn qua Presigned URLs. Dữ liệu nghiệp vụ được lưu trữ trên MongoDB Atlas, còn Groq API cung cấp khả năng xử lý Generative AI tốc độ cao. Quy trình CI/CD hoàn toàn tự động thông qua GitHub Actions kết hợp GitHub OIDC và AWS Systems Manager Parameter Store với cơ chế Instance Refresh (Zero-downtime deployment & Auto Rollback).

---

### 2. Tuyên Bố Vấn Đề

Nhiều nền tảng E-Learning truyền thống hiện nay vẫn đối mặt với các bất cập lớn:

- **Phân tán trải nghiệm**: Hoạt động học tập bị chia rẽ giữa trang xem video, nơi tải tài liệu, công cụ làm quiz và các chatbot AI độc lập không bám sát nội dung môn học.
- **Quá tải vận hành thủ công**: Giảng viên mất nhiều thời gian đọc tài liệu, chuẩn bị bản tóm tắt, biên soạn câu hỏi quiz và theo dõi tiến độ từng học viên.
- **Xử lý tài liệu & Media kém hiệu quả**: Tài liệu PDF, DOCX và PDF scan chưa được tự động trích xuất ngữ cảnh/OCR. Việc truyền file video lớn trực tiếp qua Backend gây nghẽn băng thông và treo server.
- **Rủi ro hạ tầng đơn lẻ (Single Point of Failure - SPOF)**: Triển khai backend trên 1 server EC2 đơn lẻ dễ dẫn đến ngưng trệ hệ thống khi server gặp sự cố hoặc khi lưu lượng truy cập tăng đột biến.
- **Rủi ro triển khai & Bảo mật**: Triển khai thủ công, dùng access key dài hạn hoặc lộ thông tin credentials gây gián đoạn dịch vụ và nguy cơ mất an toàn thông tin.

LearnSphere giải quyết triệt để các vấn đề trên bằng hệ thống thống nhất, phân quyền rõ ràng, lưu trữ media an toàn, tích hợp AI bám sát tài liệu và kiến trúc High Availability Multi-AZ tự động hóa 100% trên AWS.

---

### 3. Mục Tiêu Và Phạm Vi

#### 3.1. Mục Tiêu Hệ Thống

- Triển khai SPA hoàn chỉnh truy cập an toàn qua **`https://www.learnspherev2.id.vn`** với HTTPS chuẩn hóa.
- Hỗ trợ 3 vai trò `student`, `tutor` và `admin` với phân quyền chặt chẽ.
- Nâng cấp Backend lên kiến trúc High Availability (HA) Multi-AZ, đứng sau Application Load Balancer và Auto Scaling Group (tối thiểu 2 instances nằm trong Private Subnets).
- Bảo mật tài nguyên lưu trữ với S3 Private Buckets (Frontend tĩnh & Media) kết hợp Origin Access Control (OAC) và Presigned Multipart Uploads.
- Tích hợp AI (Groq API, OpenAI API) xử lý chat theo ngữ cảnh, tóm tắt document, OCR PDF scan tiếng Việt và tự động sinh quiz.
- Xây dựng pipeline CI/CD tự động 100% bằng GitHub Actions và GitHub OIDC, triển khai theo cơ chế Instance Refresh không gây gián đoạn dịch vụ (Zero downtime) và tự động Rollback khi kiểm thử thất bại.
- Giám sát toàn diện hạ tầng và log ứng dụng qua Amazon CloudWatch, gửi cảnh báo tức thì qua Amazon SNS khi có sự cố.

#### 3.2. Phạm Vi Chức Năng

| Vai trò | Chức năng chính |
| --- | --- |
| **Học viên** | Đăng ký khóa học, xem bài học/video, đọc tài liệu, theo dõi tiến độ học tập, làm quiz trắc nghiệm, nhận phản hồi AI Assistant, tham gia thảo luận khóa học. |
| **Giảng viên** | Tạo & quản lý khóa học/bài học/media; duyệt học viên; sử dụng AI để tóm tắt tài liệu và sinh câu hỏi quiz; theo dõi báo cáo chi tiết lượt làm quiz của học viên. |
| **Quản trị viên** | Quản lý tài khoản & phân quyền người dùng; kiểm duyệt khóa học; theo dõi thống kê hệ thống, dung lượng S3 và giám sát log CloudWatch. |
| **Nền tảng** | JWT Authentication (HttpOnly Secure Cookie), S3 Presigned/Multipart Upload, Orphan cleanup, AI Rate limit, GitHub OIDC CI/CD, Multi-AZ ASG & ALB Health Check. |

---

### 4. Kiến Trúc Hệ Thống

![Sơ đồ kiến trúc Production High Availability LearnSphere](/images/LEARNSHPHERE.png)

**Hình 1. Kiến trúc Production High Availability và luồng tương tác dịch vụ LearnSphere.**

> **Định hướng phát triển kiến trúc tương lai (AWS Native Roadmap):**  
> Trong các phiên bản tiếp theo, nhóm định hướng chuyển đổi **MongoDB Atlas sang Amazon DynamoDB** (sử dụng thư viện ODM `Dynamoose` giúp kết nối nội bộ qua VPC Gateway Endpoints) và chuyển **Groq API sang Amazon Bedrock** (truy cập trực tiếp Claude 3.5, tích hợp sẵn Bedrock Guardrails & Knowledge Bases cho RAG). Đồng thời, tích hợp **AWS WAF** ở vòng ngoài và sử dụng VPC Interface Endpoints để tối ưu bảo mật và chi phí hạ tầng.

#### 4.1. Luồng Truy Cập Ứng Dụng (User Request Flow)

1. Trình duyệt người dùng phân giải tên miền `https://www.learnspherev2.id.vn` qua DNS TenTen tới Amazon CloudFront CDN.
2. Kết nối HTTPS mã hóa an toàn sử dụng chứng chỉ TLS/SSL do AWS Certificate Manager (ACM) quản lý.
3. CloudFront lấy tài nguyên Frontend tĩnh từ Amazon S3 Bucket Frontend thông qua Origin Access Control (OAC).
4. Các API request dạng `/api/*` được CloudFront chuyển tiếp trực tiếp (cache disabled) tới Internet-facing Application Load Balancer qua HTTPS port 443 (hoặc `origin.learnspherev2.id.vn`).
5. ALB điều hướng traffic cân bằng tới các EC2 Backend Instances nằm trong 2 Private Subnet thuộc 2 Availability Zones (`ap-southeast-1a` và `ap-southeast-1b`).
6. EC2 Backend xử lý business logic, xác thực JWT, truy vấn MongoDB Atlas và tương tác với Groq/OpenAI API thông qua 2 NAT Gateways độc lập theo từng AZ.
7. Với file media (video, PDF, thumbnail), Backend tạo Presigned URL có thời hạn để trình duyệt upload/download trực tiếp với Amazon S3 Media Bucket.

#### 4.2. Luồng CI/CD & Deploy Tự Động (Automation Flow)

1. Developer push code mới lên nhánh `main` của repository GitHub.
2. GitHub Actions xác thực với AWS qua GitHub OIDC (không lưu access key dài hạn), nhận credentials tạm thời từ IAM Role.
3. Workflow tự động chạy Unit Test, build Docker Image theo Git SHA và push lên Amazon ECR.
4. Cập nhật tag image mới vào AWS Systems Manager (SSM) Parameter Store.
5. Kích hoạt Auto Scaling Instance Refresh: ASG khởi tạo các EC2 mới với Launch Template mới, thực hiện health check `/health/ready` qua Target Group. Khi máy mới khỏe mạnh mới tiến hành hủy máy cũ (launch-before-terminate). Nếu thất bại, tự động rollback về image cũ.
6. Build Frontend React, đồng bộ file tĩnh lên Amazon S3 Bucket Frontend và thực hiện Invalidate CloudFront Cache.

---

### 5. Thiết Kế Thành Phần Kỹ Thuật

| Thành phần | Dịch vụ / Công nghệ | Vai trò kỹ thuật trong hệ thống |
| --- | --- | --- |
| **Frontend** | React 18, TypeScript, Vite, Tailwind CSS | Chạy phía Client SPA, giao diện theo vai trò, upload trực tiếp S3, làm quiz và chat AI. |
| **Backend** | Node.js, Express 5, Docker | REST API, authentication JWT, business logic, presigned URL, AI orchestration, health check `/health/ready`. |
| **Compute / HA** | Amazon EC2 (`t3.small`), Auto Scaling Group, ALB | Vận hành Docker container trên 2 AZs trong Private Subnet, tự động mở rộng (2-4 instances) và cân bằng tải. |
| **Container Registry** | Amazon ECR | Lưu trữ các phiên bản Docker Image bất biến theo commit Git SHA. |
| **Database** | MongoDB Atlas, Mongoose ODM | Lưu trữ thông tin User, Course, Lesson, Enrollment, Quiz, Attempt, Progress, Notification, AI Message. |
| **Storage** | 2 Private Amazon S3 Buckets | Amazon S3 Frontend (lưu static build) & Amazon S3 Media (lưu video, PDF, DOCX, avatar). |
| **CDN & DNS** | Amazon CloudFront, TenTen DNS, ACM | CDN caching toàn cầu, quản lý tên miền `www.learnspherev2.id.vn` và cấp chứng chỉ HTTPS SSL. |
| **Networking** | AWS VPC, 2 Public Subnets, 2 Private Subnets, 2 NAT Gateways | Phân vùng mạng an toàn Multi-AZ, cách ly EC2 Backend khỏi Internet và cung cấp Outbound qua NAT. |
| **Configuration** | AWS SSM Parameter Store | Quản lý biến môi trường mã hóa `.env` và Docker image tag production tập trung. |
| **CI/CD** | GitHub Actions, AWS OIDC, Instance Refresh | Tự động hóa test, build, push ECR, deploy zero-downtime và auto-rollback khi có lỗi. |
| **Monitoring** | Amazon CloudWatch, Amazon SNS | Thu thập log container, cảnh báo chỉ số CPU/ALB Health và gửi thông báo khẩn qua Email. |

---

### 6. Triển Khai Kỹ Thuật & Bảo Mật

- **Phân vùng mạng an toàn (Network Segmentation)**: EC2 Backend nằm hoàn toàn trong Private Subnets, không có Public IP trực tiếp. Mọi kết nối đi vào phải thông qua ALB ở Public Subnets.
- **Xác thực OIDC & IAM Least Privilege**: Sử dụng GitHub OIDC cấp quyền tạm thời cho CI/CD. EC2 gán IAM Role chỉ có quyền đọc SSM, ghi CloudWatch Log và thao tác S3 media bucket.
- **Bảo mật dữ liệu & Storage**: S3 Buckets bật *Block Public Access*, sử dụng Origin Access Control (OAC) cho CloudFront. Media chỉ chia sẻ qua Presigned URLs ngắn hạn (15-60 phút).
- **Quản lý Secrets tập trung**: Không hardcode secret hay API keys trong source code; toàn bộ cấu hình production được mã hóa lưu trữ trên SSM Parameter Store và nạp vào container lúc runtime.
- **Kiểm soát Outbound & Whitelisting**: Mỗi AZ được trang bị một NAT Gateway riêng biệt. MongoDB Atlas cấu hình IP Whitelisting chỉ chấp nhận kết nối từ Elastic IP của các NAT Gateways trên AWS.

---

### 7. Kế Hoạch Triển Khai & Mốc Công Việc

| Giai đoạn | Thời gian | Nội dung triển khai | Mốc kết quả bàn giao |
| --- | --- | --- | --- |
| **1. Nền tảng AWS & VPC** | Tuần 1–4 | Thiết lập VPC Multi-AZ, Public/Private Subnets, NAT Gateways, ECR, S3 Buckets, ACM Certificate và OIDC Provider. | Hạ tầng Cloud sẵn sàng. |
| **2. Backend & Containerization** | Tuần 5 | Đóng gói Docker Backend Node.js, cài đặt health check `/health/ready`, tích hợp MongoDB Atlas & S3 Presigned URLs. | API nghiệp vụ sẵn sàng dạng Docker Image. |
| **3. Tích hợp AI & Document OCR** | Tuần 6 | Tích hợp Groq API, xử lý trích xuất văn bản PDF/DOCX, OCR Tesseract.js cho PDF scan, tính năng AI Tutor & Quiz Generator. | Tính năng AI bám sát tài liệu hoạt động mượt mà. |
| **4. Frontend & Testing** | Tuần 7 | Xây dựng giao diện React/Vite responsive, quản lý khóa học, tiến độ, quiz, tích hợp CloudFront CDN & S3 OAC. | Hoàn thiện trải nghiệm người dùng trên SPA. |
| **5. Triển khai Production HA** | Tuần 8 | Cấu hình ALB, Launch Template, Auto Scaling Group Multi-AZ, SSM Parameter Store, GitHub Actions CI/CD với Instance Refresh. | Hệ thống LearnSphere chạy HA trên tên miền `www.learnspherev2.id.vn`. |
| **6. Nghiệm thu & Báo cáo** | Tuần 9 | End-to-end testing, kiểm thử chịu tải, kiểm tra auto-rollback, hoàn thiện bộ tài liệu báo cáo thực tập & Workshop. | Bàn giao báo cáo nghiệm thu hoàn chỉnh. |

---

### 8. Ước Tính Ngân Sách Vận Hành (Cost Analysis)

Để tối ưu hóa chi phí và đảm bảo hiệu quả ngân sách cho dự án LearnSphere trong giai đoạn workshop, hệ thống áp dụng kịch bản vận hành Multi-AZ có kiểm soát tại AWS Region Singapore (`ap-southeast-1`). Tổng chi phí ước tính được tóm tắt như sau:

- **Tạm tính hạ tầng chính (Compute, Network, Storage, Load Balancer)**: 53,42 USD/tháng (bao gồm 2 EC2 `t3.small`, 2 NAT Gateways, 1 ALB, S3 Standard, ECR và CloudWatch Logs running 240h/tháng).
- **Dự phòng biến động (5%)**: 2,67 USD/tháng (dành cho lưu lượng data transfer, request và log phát sinh thêm).
- **TỔNG DỰ TOÁN VẬN HÀNH WORKSHOP**: 56,09 USD/tháng (mức chi phí mục tiêu nằm trong khoảng 50–60 USD/tháng).

 
Mô hình ước tính này đảm bảo sự cân bằng giữa tính sẵn sàng cao (High Availability Multi-AZ) và việc kiểm soát chi phí thực tập hiệu quả. Chi phí phần lớn tập trung vào NAT Gateway và ALB để duy trì hạ tầng mạng an toàn trong Private Subnet. Chi phí chi tiết theo từng hạng mục tài nguyên và các giải pháp tối ưu ngân sách nâng cao được phân tích cụ thể tại Mục 5.11. Phân tích chi phí.

---

### 9. Đánh Giá Rủi Ro & Giải Pháp Tối Ưu

| Rủi ro hệ thống | Mức độ | Khả năng | Biện pháp giảm thiểu & Thiết kế hạ tầng |
| --- | --- | --- | --- |
| **EC2 Instance bị sự cố** | Thấp | Trung bình | Auto Scaling Group tự động thay thế máy lỗi dựa trên ALB/EC2 Health Checks. |
| **Deploy code bị lỗi ngắt dịch vụ** | Trung bình | Thấp | Cơ chế Instance Refresh (Launch-before-terminate) đảm bảo zero downtime & auto-rollback. |
| **Chi phí NAT Gateway cao** | Trung bình | Cao | Dùng S3 Gateway Endpoint miễn phí cho traffic S3; tương lai dùng VPC Endpoints cho dịch vụ AWS. |
| **Lỗi nghẽn bộ nhớ khi OCR PDF scan** | Trung bình | Trung bình | Giới hạn dung lượng file, chạy OCR tuần tự trên worker riêng, thiết lập timeout & memory limits. |
| **Rò rỉ credentials / Secrets** | Cao | Thấp | Sử dụng GitHub OIDC cho CI/CD, SSM Parameter Store mã hóa secrets, không lưu access key dài hạn. |

---

### 10. Kết Quả Đạt Được & Giá Trị Vận Hành

- **Truy cập an toàn & Tốc độ cao**: Ứng dụng vận hành hoàn chỉnh trên tên miền **`https://www.learnspherev2.id.vn`**, mã hóa HTTPS 100%, Frontend đạt tốc độ tải tức thì nhờ CloudFront Edge Caching.
- **Hạ tầng sẵn sàng cao (High Availability)**: Backend được bảo vệ trong Private Subnets, tự động cân bằng tải qua ALB và mở rộng linh hoạt với Auto Scaling Group trên 2 AZs.
- **Tự động hóa 100% CI/CD**: Quá trình chuyển giao mã nguồn từ GitHub lên AWS diễn ra hoàn toàn tự động, an toàn với OIDC, không downtime và hỗ trợ quay về phiên bản cũ tức tính khi có lỗi.
- **Tích hợp AI thông minh**: Trợ lý AI Assistant phản hồi bám sát tài liệu học tập, hỗ trợ tóm tắt bài học và sinh câu hỏi quiz hiệu quả, tiết kiệm 70% thời gian chuẩn bị học liệu cho giảng viên.
