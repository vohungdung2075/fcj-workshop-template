---
title: "Kiến trúc và luồng hệ thống"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Mục tiêu thiết kế

Kiến trúc production của LearnSphere được xây dựng theo mô hình nhiều lớp nhằm tách biệt giao diện, API, dữ liệu và các dịch vụ tích hợp. Thiết kế hướng đến các mục tiêu chính:

* Cung cấp Frontend qua HTTPS với độ trễ thấp và hỗ trợ điều hướng SPA.
* Không để Backend EC2 hoặc các S3 bucket truy cập công khai trực tiếp.
* Duy trì tối thiểu hai Backend instance tại hai Availability Zone.
* Tách dữ liệu trạng thái khỏi EC2 để instance có thể được thay thế an toàn.
* Upload video, tài liệu và thumbnail trực tiếp đến S3, tránh truyền file lớn qua Backend.
* Tự động hóa kiểm thử, đóng gói và triển khai bằng GitHub Actions.
* Tập trung log, metric, health check và cảnh báo phục vụ vận hành.

Kiến trúc được triển khai tại Region Singapore (`ap-southeast-1`). Chứng chỉ dùng cho CloudFront được tạo tại `us-east-1` theo yêu cầu của dịch vụ, còn chứng chỉ của Application Load Balancer được tạo tại `ap-southeast-1`.

#### Sơ đồ kiến trúc

![Kiến trúc production LearnSphere Multi-AZ](/images/LEARNSHPHERE.png)

*Hình 5.6. Kiến trúc As-Built của LearnSphere sau khi nâng cấp Backend theo mô hình High Availability Multi-AZ.*
> **Định hướng phát triển kiến trúc tương lai (AWS Native Roadmap):**  
> Trong các phiên bản tiếp theo, nhóm định hướng chuyển đổi **MongoDB Atlas sang Amazon DynamoDB** (sử dụng thư viện ODM `Dynamoose` giúp kết nối nội bộ qua VPC Gateway Endpoints) và chuyển **Groq API sang Amazon Bedrock** (truy cập trực tiếp Claude 3.5, tích hợp sẵn Bedrock Guardrails & Knowledge Bases cho RAG). Đồng thời, tích hợp **AWS WAF** ở vòng ngoài và sử dụng VPC Interface Endpoints để tối ưu bảo mật và chi phí hạ tầng.

Sơ đồ thể hiện hai nhóm luồng:
* **Đường liền** biểu diễn traffic runtime và luồng dữ liệu của người dùng, Backend hoặc dịch vụ giám sát.
* **Đường nét đứt** biểu diễn luồng quản trị, cấp quyền, cấu hình và triển khai từ GitHub Actions.

#### Phân lớp kiến trúc

| Lớp | Thành phần | Vai trò |
| --- | --- | --- |
| Client và DNS | Trình duyệt, DNS TenTen | Phân giải `www.learnspherev2.id.vn` và gửi request HTTPS |
| Edge | Amazon CloudFront, AWS Certificate Manager | Phân phối Frontend, kết thúc TLS và định tuyến `/api/*` |
| Frontend | Amazon S3 Frontend | Lưu bản build tĩnh React/Vite; bucket private và chỉ cho CloudFront OAC đọc |
| Network | VPC, public/private subnet, Internet Gateway, NAT Gateway, S3 Gateway Endpoint | Cô lập Backend và cung cấp đường vào/ra có kiểm soát |
| API entry | Application Load Balancer | Nhận HTTPS từ CloudFront và phân phối request đến Target Group |
| Compute | Auto Scaling Group, Launch Template, EC2, Docker | Chạy Express.js Backend trên port 5000 và tự thay instance lỗi |
| Media | Amazon S3 Media | Lưu video, document, thumbnail và avatar qua presigned URL |
| Configuration | AWS Systems Manager Parameter Store | Lưu environment production dạng SecureString và image tag đang phát hành |
| Container registry | Amazon ECR | Lưu Docker image Backend theo immutable commit SHA |
| Data | MongoDB Atlas | Lưu user, course, lesson, quiz, enrollment, progress và dữ liệu AI |
| AI | Groq API | Thực hiện chat, tóm tắt tài liệu và sinh câu hỏi quiz |
| Observability | Amazon CloudWatch và Amazon SNS | Thu thập log/metric, đánh giá alarm và gửi thông báo email |
| Delivery | GitHub Actions và IAM OIDC | Kiểm thử, build và triển khai mà không dùng Access Key dài hạn |

#### Kiến trúc mạng Multi-AZ

LearnSphere sử dụng VPC `10.20.0.0/16` với DNS resolution và DNS hostnames được bật. Tài nguyên được phân bố trên hai Availability Zone:

| Availability Zone | Public subnet | Private subnet | Tài nguyên chính |
| --- | --- | --- | --- |
| `ap-southeast-1a` | Public subnet 1a | Private subnet 1a | ALB node, NAT Gateway 1a và Backend EC2 1a |
| `ap-southeast-1b` | Public subnet 1b | Private subnet 1b | ALB node, NAT Gateway 1b và Backend EC2 1b |

Hai public subnet có route `0.0.0.0/0` đến Internet Gateway. Mỗi private subnet có route mặc định đến NAT Gateway trong cùng Availability Zone. Cách bố trí này tránh trường hợp Backend ở một AZ phụ thuộc vào NAT Gateway của AZ còn lại.

Application Load Balancer được gắn vào hai public subnet để có thể tiếp nhận HTTPS. Các EC2 Backend chỉ nằm trong private subnet, không có public IPv4 và không mở SSH. EC2 vẫn có thể tải Docker image, gọi API bên ngoài và gửi email thông qua NAT Gateway.

S3 Gateway Endpoint được gắn vào hai private route table. Vì vậy, request từ EC2 đến Amazon S3 đi trong mạng AWS thay vì đi qua NAT Gateway, giúp giảm traffic Internet và chi phí NAT data processing.

![VPC Resource Map của LearnSphere](/images/learnsphere-vpc-resource-map.png)

*Hình 5.7. VPC Resource Map thể hiện bốn subnet trên hai Availability Zone, các route table, Internet Gateway, hai NAT Gateway và S3 Gateway Endpoint của LearnSphere.*

#### Luồng truy cập Frontend và API

Luồng tải ứng dụng và gọi API được xử lý như sau:

1. Người dùng truy cập `https://www.learnspherev2.id.vn`.
2. DNS TenTen trả về domain của CloudFront distribution.
3. CloudFront sử dụng chứng chỉ ACM để thiết lập kết nối HTTPS.
4. Với request giao diện, default behavior đọc `index.html` và static assets từ S3 Frontend private qua Origin Access Control.
5. CloudFront Function rewrite các route SPA như `/profile`, `/courses` hoặc `/system-monitoring` về `index.html`.
6. Khi Frontend gọi `/api/*`, CloudFront chuyển request qua HTTPS đến origin `origin.learnspherev2.id.vn`.
7. Origin domain trỏ đến `LearnSphere-Prod-ALB`.
8. ALB kết thúc TLS, kiểm tra rule và chuyển request HTTP port 5000 đến `LearnSphere-Backend-TG`.
9. Target Group chỉ gửi request đến các EC2 có trạng thái healthy trong hai private subnet.
10. Backend xác thực JWT/cookie, thực thi nghiệp vụ và truy cập MongoDB Atlas hoặc dịch vụ liên quan.
11. Response quay lại theo chiều EC2 → ALB → CloudFront → trình duyệt.

CloudFront trở thành điểm truy cập thống nhất cho cả giao diện và API. Cách tổ chức này giúp Frontend gọi API bằng cùng origin `/api/*`, giảm cấu hình CORS và tránh để người dùng phải biết DNS trực tiếp của ALB.

#### Luồng upload và tải học liệu

Video và tài liệu có thể có dung lượng lớn nên LearnSphere không truyền nội dung file qua Express.js. Luồng presigned URL được áp dụng:

1. Người dùng đã đăng nhập yêu cầu Backend tạo upload session.
2. Backend kiểm tra role, quyền sở hữu khóa học, loại file, dung lượng và S3 object key.
3. Backend dùng EC2 instance role để tạo presigned URL hoặc multipart presigned URLs.
4. Trình duyệt upload trực tiếp file đến S3 Media.
5. Sau khi upload thành công, Frontend gọi API hoàn tất session.
6. Backend kiểm tra object trên S3 rồi lưu key và metadata vào MongoDB.
7. Khi xem bài học, Backend tạo presigned download URL có thời hạn và trả URL cho người dùng có quyền.

Luồng này giảm tải CPU, RAM và băng thông trên EC2. S3 Media vẫn bật Block Public Access; quyền truy cập được giới hạn theo thời gian và theo object thông qua chữ ký của Backend. Cơ chế cleanup xử lý multipart session hết hạn và object mồ côi nhằm hạn chế phát sinh chi phí lưu trữ.

#### Luồng dữ liệu và xử lý AI

MongoDB Atlas là nguồn dữ liệu dùng chung cho mọi Backend instance. EC2 không lưu dữ liệu nghiệp vụ lâu dài trên ổ đĩa cục bộ, vì vậy ASG có thể terminate hoặc thay instance mà không làm mất course, lesson, quiz hay progress.

Khi giáo viên yêu cầu tóm tắt document hoặc sinh quiz:

1. Backend lấy metadata tài liệu từ MongoDB.
2. Backend đọc document tương ứng từ S3 Media.
3. Nội dung được trích xuất bằng `pdf-parse`, `mammoth` hoặc OCR bằng `tesseract.js` đối với PDF scan.
4. Văn bản được giới hạn và chuẩn hóa trước khi gửi đến Groq API.
5. Kết quả AI được kiểm tra cấu trúc, sau đó trả về Frontend hoặc lưu vào MongoDB theo chức năng.
6. `model_id`, input token và output token được ghi nhận để hỗ trợ theo dõi mức sử dụng.

Chat của học viên cũng đi qua Backend để thực hiện xác thực, rate limiting, chọn ngữ cảnh bài học và lưu lịch sử. API key của Groq không bao giờ được gửi xuống trình duyệt.

Backend tại AZ 1a đi Internet bằng NAT Gateway 1a, còn Backend tại AZ 1b dùng NAT Gateway 1b. Hai Elastic IP của NAT được allowlist trong MongoDB Atlas Network Access. Groq API và dịch vụ email được gọi qua kết nối TLS từ các địa chỉ egress này.

#### Luồng cấu hình và khởi tạo EC2

Launch Template chứa AMI, instance type, Backend Security Group, EC2 instance profile và User Data. Khi ASG tạo một instance mới, quá trình bootstrap thực hiện:

1. Cài đặt và khởi động Docker.
2. Đọc environment production từ `/learnsphere/prod/backend-env` trong Parameter Store.
3. Đọc image tag hiện hành từ `/learnsphere/prod/backend-image-tag`.
4. Đăng nhập ECR bằng quyền tạm thời của EC2 instance role.
5. Pull đúng Docker image đã được pipeline phát hành.
6. Chạy container với CloudWatch Logs driver.
7. Gọi `/health/ready` và chỉ hoàn tất khi MongoDB đã kết nối.

Environment được lưu dưới dạng SSM SecureString, còn image tag được tách thành String riêng. GitHub Actions không cần đọc nội dung secret production; EC2 tự lấy cấu hình bằng instance role với quyền tối thiểu.

#### Luồng CI/CD

Mỗi lần triển khai, GitHub Actions lấy temporary credentials bằng OIDC và IAM role thay vì lưu `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY` dài hạn.

```text
GitHub Actions
  → Assume IAM role bằng OIDC
  → chạy Backend tests
  → build Docker image
  → push image với tag commit SHA lên ECR
  → cập nhật candidate image tag trong Parameter Store
  → khởi chạy ASG Instance Refresh
  → chờ EC2 và Target Group healthy
  → kiểm tra /health/ready qua ALB
  → build Frontend React/Vite
  → đồng bộ dist lên S3 Frontend
  → tạo CloudFront invalidation
```

Instance Refresh dùng chiến lược launch-before-terminate. Instance mới phải vượt qua health check trước khi instance cũ bị loại bỏ. Nếu candidate không healthy, workflow dừng, khôi phục image tag trước đó và giữ năng lực phục vụ hiện tại.

#### High Availability và khả năng tự phục hồi

`LearnSphere-Backend-ASG` có cấu hình:

| Thuộc tính | Giá trị |
| --- | --- |
| Minimum capacity | 2 |
| Desired capacity | 2 |
| Maximum capacity | 4 |
| Health checks | EC2 và ELB |
| Phân bố | Hai private subnet thuộc hai AZ |
| Maintenance | Launch before terminating |

![Target Group Backend của LearnSphere ở trạng thái healthy](/images/learnsphere-target-group-healthy.png)

*Hình 5.8. Target Group của Application Load Balancer ghi nhận hai Backend target healthy trên port 5000 và không có target unhealthy.*

Trong trạng thái bình thường, mỗi AZ có một Backend EC2. Nếu một container, instance hoặc AZ target gặp lỗi:

1. Readiness check thất bại và ALB ngừng định tuyến đến target lỗi.
2. Request tiếp tục được gửi đến target healthy còn lại.
3. ASG nhận kết quả ELB health check và tạo instance thay thế.
4. Instance mới bootstrap từ cùng Launch Template, Parameter Store và ECR.
5. Khi target mới healthy, ASG trở lại trạng thái desired capacity.

ALB và ASG loại bỏ single point of failure tại tầng Backend. S3 và CloudFront là dịch vụ managed có tính sẵn sàng cao. Hai NAT Gateway cung cấp egress độc lập theo AZ.

Giá trị `max=4` là giới hạn capacity, không tự động đồng nghĩa với scale-out. Kiến trúc hiện duy trì và tự phục hồi hai instance; để tự động tăng/giảm theo tải cần bổ sung target tracking hoặc step scaling policy dựa trên CPU, request count hoặc response time.

#### Ranh giới bảo mật

| Ranh giới | Cấu hình đang áp dụng |
| --- | --- |
| HTTPS | ACM cho CloudFront tại `us-east-1` và ACM cho ALB tại `ap-southeast-1` |
| ALB Security Group | Nhận HTTPS port 443; chuyển tiếp đến Backend Target Group |
| Backend Security Group | Chỉ nhận TCP port 5000 từ ALB Security Group |
| EC2 | Private subnet, không public IP, không inbound SSH |
| S3 Frontend | Block Public Access và chỉ cho CloudFront OAC đọc |
| S3 Media | Block Public Access; upload/download bằng presigned URL |
| IAM | Tách GitHub deploy role, EC2 instance role và service-linked role của ASG |
| Secret | Environment production trong SSM SecureString; không commit `.env` |
| Database | MongoDB Atlas chỉ allowlist hai NAT Elastic IP |
| Application | CORS theo HTTPS origin, JWT secret, security headers và rate limiting |

#### Giám sát và cảnh báo

Container Backend gửi stdout/stderr đến CloudWatch Logs. Liveness endpoint cho biết tiến trình Node.js còn hoạt động, còn readiness endpoint chỉ trả `ready` khi kết nối MongoDB sẵn sàng. ALB dùng readiness endpoint để quyết định target có được nhận traffic hay không.

CloudWatch cung cấp metric của EC2, ASG và ALB. Alarm có thể theo dõi số target unhealthy, số instance InService, HTTP 5xx, response time và CPU. Khi alarm chuyển trạng thái, Amazon SNS gửi email cho quản trị viên qua topic `LearnSphere-Alerts`.

#### Đánh giá theo AWS Well-Architected Framework

| Trụ cột | Cách LearnSphere đáp ứng |
| --- | --- |
| Operational Excellence | CI/CD tự động, immutable image, health check, centralized logs và quy trình rollback |
| Security | HTTPS end-to-end, private subnet, OAC, presigned URL, IAM role và SSM SecureString |
| Reliability | Hai AZ, ALB, ASG self-healing, hai NAT Gateway và state nằm ngoài EC2 |
| Performance Efficiency | CloudFront cache static assets, direct S3 upload và S3 Gateway Endpoint |
| Cost Optimization | AWS Budget, S3 lifecycle/cleanup, endpoint tránh NAT cho S3 và tài nguyên có giới hạn capacity |
| Sustainability | Static hosting, CDN caching, container có thể thay thế và giảm truyền file qua compute |

#### Phạm vi và giới hạn hiện tại

High Availability hiện được triển khai trong một AWS Region với hai Availability Zone; đây chưa phải kiến trúc Disaster Recovery đa Region. MongoDB Atlas và Groq là dependency bên ngoài AWS Account, do đó mức HA cuối cùng còn phụ thuộc cấu hình cluster Atlas và khả năng cung cấp của Groq.

Trong giai đoạn phát triển tiếp theo, LearnSphere định hướng chuyển lớp dữ liệu từ MongoDB Atlas sang Amazon DynamoDB để đưa dữ liệu nghiệp vụ vào hệ sinh thái AWS, tận dụng khả năng mở rộng serverless và loại bỏ công việc quản trị máy chủ cơ sở dữ liệu. Quá trình này cần được thực hiện theo từng giai đoạn: phân tích access pattern, thiết kế partition key và sort key, thay thế lớp Mongoose, xây dựng công cụ migration, kiểm thử tính nhất quán và chỉ chuyển traffic sau khi dữ liệu đã được xác minh.

Lớp AI cũng được định hướng chuyển từ Groq sang Amazon Bedrock để giảm phụ thuộc vào nhà cung cấp bên ngoài, thống nhất cơ chế phân quyền bằng IAM và đưa hoạt động suy luận AI vào cùng hệ sinh thái giám sát, bảo mật và quản trị chi phí của AWS. Quá trình chuyển đổi sẽ tận dụng lớp abstraction nhà cung cấp AI hiện có, sau đó cấu hình model phù hợp trên Bedrock, cấp quyền invoke theo nguyên tắc least privilege, kiểm tra service quota và đối chiếu chất lượng câu trả lời, độ trễ, số token và chi phí với Groq. Groq chỉ được loại bỏ sau khi các chức năng trợ lý học tập, tóm tắt tài liệu và sinh câu hỏi đã vượt qua kiểm thử hồi quy trên Bedrock.

Hạ tầng cũng sẽ được mở rộng bằng Amazon Route 53 để quản lý DNS và health check tập trung; AWS WAF tại CloudFront để lọc request độc hại, rate-based attack và các mẫu khai thác phổ biến; ASG target tracking để tự động mở rộng theo tải; AWS Secrets Manager để quản lý và xoay vòng secret; cùng phương án backup, restore và failover đa Region. Các cải tiến này là lộ trình phát triển từ kiến trúc Multi-AZ hiện tại đến một nền tảng cloud-native có khả năng bảo mật, mở rộng và phục hồi cao hơn.
