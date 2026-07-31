---
title: "Bản đề xuất"
date: 2026-06-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# LearnSphere - Online Learning Platform Integrated with AI
## Giải pháp Học tập Hiện đại trên Nền tảng Đám mây AWS

### 1. Tóm tắt điều hành

LearnSphere được đề xuất là một nền tảng học tập trực tuyến dành cho học viên, giảng viên và quản trị viên. Hệ thống tập trung khóa học, bài học, video, tài liệu, quiz, tiến độ học tập và trao đổi trong cùng một ứng dụng. AI Assistant hỗ trợ học viên đặt câu hỏi dựa trên nội dung bài học, tóm tắt tài liệu và hỗ trợ giảng viên sinh câu hỏi quiz theo mức độ khó.

Giải pháp sử dụng kiến trúc web kết hợp trên AWS. Frontend React/Vite được build thành tài nguyên tĩnh, lưu trong private Amazon S3 bucket và phân phối qua Amazon CloudFront. Tên miền `www.learnsphere.id.vn` được quản lý DNS tại Tenten, trỏ CNAME tới CloudFront và sử dụng chứng chỉ HTTPS do AWS Certificate Manager cấp. Backend Express.js được đóng gói bằng Docker, lưu tại Amazon ECR và chạy trên Amazon EC2. Media của khóa học được lưu trong một private S3 bucket khác và chỉ được truy cập qua presigned URL có thời hạn. MongoDB Atlas lưu dữ liệu nghiệp vụ, còn Groq API cung cấp khả năng suy luận AI cho chat, tóm tắt tài liệu và sinh câu hỏi quiz. GitHub Actions tự động kiểm thử, build và triển khai hệ thống bằng AWS OIDC và Systems Manager.

Phiên bản MVP ưu tiên chi phí phù hợp với dự án sinh viên, khả năng triển khai được trong thời gian thực tập, bảo mật tài nguyên và truy cập thuận tiện qua tên miền riêng có HTTPS. Khi lượng người dùng tăng, kiến trúc có thể bổ sung AWS WAF, Application Load Balancer, Auto Scaling, Secrets Manager và quy trình xử lý media bất đồng bộ.

### 2. Tuyên bố vấn đề

Nhiều hoạt động học tập trực tuyến vẫn bị phân tán giữa nền tảng xem video, nơi chia sẻ tài liệu, công cụ làm quiz và các chatbot AI độc lập. Điều này tạo ra các vấn đề:

* Học viên khó theo dõi khóa học, tiến độ và kết quả quiz tại một nơi.
* Các nền tảng E-Learning truyền thống còn hạn chế khả năng hỗ trợ theo ngữ cảnh ngoài giờ học, trong khi câu trả lời của chatbot phổ thông có thể không dựa trên tài liệu của bài học.
* Giảng viên mất nhiều thời gian đọc tài liệu, chuẩn bị bản tóm tắt, soạn câu hỏi, quản lý đăng ký và theo dõi từng học viên.
* Nội dung trong PDF, DOCX và tài liệu scan chưa được tự động chuyển thành ngữ cảnh bài học có thể tái sử dụng.
* Video và document có dung lượng lớn gây khó khăn khi upload trực tiếp qua Backend, đồng thời tiêu tốn băng thông và tài nguyên xử lý của server.
* Quản trị viên thiếu số liệu tập trung về người dùng, khóa học, lượt truy cập và dung lượng lưu trữ.
* Triển khai thủ công và log phân tán dễ gây sai cấu hình, gián đoạn dịch vụ, chậm phát hiện sự cố và khó khôi phục phiên bản trước.

LearnSphere giải quyết các vấn đề trên bằng một hệ thống thống nhất có phân quyền, lưu trữ media an toàn, AI bám theo nội dung document, quiz và báo cáo học tập, cùng quy trình triển khai và giám sát trên AWS.

### 3. Mục tiêu và phạm vi

#### 3.1. Mục tiêu

* Xây dựng một SPA responsive có thể truy cập qua `www.learnsphere.id.vn` bằng HTTPS.
* Hỗ trợ ba vai trò `student` , `tutor` và `admin` với quyền hạn rõ ràng.
* Quản lý toàn bộ vòng đời khóa học, bài học, đăng ký, tiến độ và quiz.
* Cho phép upload video lớn, document, thumbnail và avatar trực tiếp lên S3 một cách an toàn.
* Tích hợp AI để chat theo ngữ cảnh, tóm tắt document và sinh câu hỏi quiz.
* Theo dõi model, lượng token và giới hạn request AI theo người dùng.
* Tự động build, kiểm thử và triển khai từ GitHub lên AWS.
* Theo dõi trạng thái Backend, chỉ số hệ thống và gửi cảnh báo khi EC2 gặp sự cố.

#### 3.2. Phạm vi MVP

| Vai trò | Chức năng chính |
| --- | --- |
| Học viên | Đăng ký khóa học, xem bài học và tài liệu, theo dõi tiến độ, làm quiz, xem kết quả, dùng AI Assistant, nhận thông báo và tham gia thảo luận |
| Giảng viên | Tạo và quản lý khóa học, bài học, media và quiz; duyệt học viên; sinh câu hỏi bằng AI; xem tiến độ và chi tiết lượt làm quiz của học viên |
| Quản trị viên | Quản lý tài khoản và trạng thái người dùng; xem khóa học và quiz theo giảng viên; theo dõi thống kê hệ thống và dung lượng S3 |
| Nền tảng | JWT authentication, phân quyền, notification, discussion, presigned/multipart upload, dọn file rác, AI rate limit, CI/CD và monitoring |

#### 3.3. Ngoài phạm vi hiện tại

Thanh toán khóa học, lớp học trực tuyến theo thời gian thực, ứng dụng mobile native, tóm tắt nội dung video, hệ gợi ý cá nhân hóa, Multi-AZ và Auto Scaling chưa thuộc phạm vi MVP. Các nội dung này được xem là hướng phát triển tiếp theo.

### 4. Kiến trúc giải pháp đề xuất

![Sơ đồ kiến trúc hệ thống LearnSphere](/images/LEARNSHPHERE.png)

**Hình 1. Kiến trúc production và luồng tương tác dịch vụ của LearnSphere.**

Sơ đồ thể hiện các luồng vận hành, triển khai, lưu trữ, AI và giám sát. CloudFront và IAM là các dịch vụ AWS toàn cầu.\
ECR, Systems Manager, S3, EC2, CloudWatch và SNS hoạt động tại `ap-southeast-1`. \
VPC chứa Internet Gateway, một Availability Zone, public subnet và EC2 Backend. \
GitHub, người dùng, MongoDB Atlas, Groq và dịch vụ email của quản trị viên nằm ngoài AWS Cloud.

#### 4.1. Luồng truy cập ứng dụng

1. Trình duyệt phân giải `www.learnsphere.id.vn` qua DNS của Tenten tới CloudFront.
2. Người dùng kết nối HTTPS và CloudFront sử dụng chứng chỉ TLS được quản lý trong ACM.
3. CloudFront lấy Frontend từ private S3 bucket bằng Origin Access Control.
4. Request `/api/*` được chuyển đến EC2 trên port Backend và không cache.
5. Backend xác thực JWT, kiểm tra vai trò và truy vấn MongoDB Atlas.
6. Với media, Backend chỉ tạo presigned URL; trình duyệt upload hoặc download trực tiếp với private S3 bucket.

#### 4.2. Luồng AI và document

1. Giảng viên upload PDF hoặc DOCX lên S3 và gắn object key vào bài học.
2. Backend tải document theo quyền IAM, dùng `pdf-parse` hoặc Mammoth để trích xuất chữ.
3. PDF scan được OCR từng trang bằng Tesseract.js với ngôn ngữ tiếng Việt và giới hạn tài nguyên.
4. Nội dung đã chuẩn hóa được dùng cho tóm tắt, chat theo bài học hoặc sinh quiz.
5. Kết quả tóm tắt được lưu lại để tránh gọi AI lặp lại; model ID và lượng token được ghi nhận.
6. Backend gửi context đã chuẩn bị tới Groq và xử lý rõ các lỗi throttling, timeout hoặc response sai cấu trúc.

#### 4.3. Luồng CI/CD

1. Push lên nhánh `main` kích hoạt GitHub Actions.
2. Backend được cài dependency, chạy test, build Docker image và push lên ECR theo commit SHA.
3. GitHub Actions dùng OIDC nhận credential AWS tạm thời và gửi lệnh triển khai qua Systems Manager.
4. EC2 chạy candidate container, kiểm tra `/health/ready`, sau đó mới thay container production; bản cũ được giữ tạm để rollback.
5. Frontend được build, đồng bộ lên S3 và CloudFront được invalidate sau khi Backend triển khai thành công.

### 5. Thiết kế thành phần

| Thành phần | Công nghệ/Dịch vụ | Trách nhiệm |
| --- | --- | --- |
| Frontend | React 18, TypeScript, Vite, Tailwind CSS, KaTeX | SPA, giao diện theo vai trò, upload trực tiếp, quiz, AI Assistant và hiển thị công thức |
| Backend | Node.js, Express 5, Docker | REST API, authentication, business logic, signed URL, AI orchestration và health check |
| Cơ sở dữ liệu | MongoDB Atlas, Mongoose | User, Course, Lesson, Enrollment, Quiz, Attempt, Progress, Notification, AI Message và upload/cleanup state |
| Lưu trữ | Hai private Amazon S3 buckets | Frontend static assets; video, document, thumbnail và avatar |
| AI | Groq API | Chat, tóm tắt document và sinh câu hỏi |
| Xử lý document | pdf-parse, Mammoth, Tesseract.js | Trích xuất PDF/DOCX và OCR PDF scan tiếng Việt |
| Compute | Amazon EC2, Amazon Linux 2023 | Chạy Docker container Backend và tác vụ OCR |
| Container registry | Amazon ECR | Lưu Docker image theo commit SHA |
| Delivery | Amazon CloudFront và CloudFront Function | HTTPS, CDN cho SPA, route fallback và chuyển tiếp `/api/*` |
| Tên miền và TLS | Tenten DNS và AWS Certificate Manager | CNAME `www` tới CloudFront, xác thực tên miền và cung cấp chứng chỉ HTTPS |
| CI/CD | GitHub Actions, AWS OIDC, Systems Manager | Test, build, deploy, health check và rollback |
| Quan sát hệ thống | Amazon CloudWatch và SNS | Log container, CPU/status alarms và email cảnh báo |

### 6. Triển khai kỹ thuật và bảo mật

#### 6.1. Các biện pháp chính

* JWT được lưu bằng HttpOnly cookie; production sử dụng Secure cookie và CORS allowlist.
* `bcryptjs`, Helmet, rate limit, giới hạn JSON request và middleware phân quyền bảo vệ API.
* Hai S3 bucket giữ private và bật Block Public Access.
* CloudFront truy cập bucket Frontend bằng OAC; media chỉ truy cập qua presigned URL ngắn hạn.
* `www.learnsphere.id.vn` trỏ tới CloudFront bằng CNAME; ACM quản lý chứng chỉ TLS và bản ghi DNS validation được giữ lại để hỗ trợ gia hạn.
* Upload video lớn sử dụng multipart upload, theo dõi `UploadSession`, hỗ trợ abort/retry và dọn orphan object.
* Khi thay avatar, thumbnail, video hoặc document, object cũ chỉ bị xóa sau khi cập nhật database thành công.
* EC2 sử dụng IAM Role; GitHub sử dụng OIDC. Không lưu AWS Access Key dài hạn trên server hoặc repository.
* Security Group chỉ cho phép port Backend nhận traffic từ AWS managed prefix list dành cho CloudFront origin.
* AI có rate limit theo user, timeout, lỗi chuẩn hóa, cache tóm tắt và thống kê token.
* CloudWatch theo dõi `CPUUtilization` và `StatusCheckFailed`; SNS gửi email khi alarm chuyển trạng thái.

#### 6.2. Giới hạn của kiến trúc MVP

Backend hiện chạy trên một EC2 instance trong một Availability Zone nên vẫn là single point of failure. Secret ứng dụng được bảo vệ trong file `.env` quyền `600`, nhưng chưa được quản lý tập trung bằng Secrets Manager. MongoDB Atlas và Groq là dịch vụ ngoài AWS nên phụ thuộc kết nối Internet và chính sách của nhà cung cấp.

### 7. Kế hoạch triển khai và mốc công việc

| Giai đoạn | Thời gian | Nội dung | Mốc kết quả |
| --- | --- | --- | --- |
| 1. Nền tảng AWS | Tuần 1–4 | Cloud, IAM, chi phí, VPC, EC2, S3, database, CloudFront, Docker, ECR, CloudWatch, SNS, Generative AI và CI/CD | Hoàn thành kiến thức và kiến trúc cơ sở |
| 2. Backend LearnSphere | Tuần 5 | Authentication, course, lesson, enrollment, quiz, progress, notification và S3 cleanup | API nghiệp vụ chính hoàn chỉnh |
| 3. AI và document | Tuần 6 | AI Provider, chat history, rate limit, PDF/DOCX indexing, OCR, summary, AI quiz và multipart cleanup | AI bám document và upload ổn định |
| 4. Frontend và kiểm thử | Tuần 7 | Giao diện theo vai trò, course/lesson/quiz management, report học viên và kiểm thử | Hoàn chỉnh trải nghiệm người dùng |
| 5. Triển khai production | Tuần 8 | S3, CloudFront, ECR, EC2, IAM, OIDC, CI/CD, CloudWatch, SNS, ACM và custom domain | LearnSphere hoạt động trên AWS qua `www.learnsphere.id.vn` |
| 6. Nghiệm thu và báo cáo | Tuần 9 | End-to-end test, hình ảnh, kiến trúc, workshop song ngữ và demo | Hoàn thành bàn giao và báo cáo |

### 8. Ước tính ngân sách

Giả định: một EC2 `t3.small` chạy 24/7, khoảng 8–10 GB EBS, 10 GB media S3, dưới 100 GB dữ liệu phân phối mỗi tháng, dưới 1 GB image ECR, lượng log thấp và quy mô thử nghiệm khoảng 50 người dùng.

| Thành phần | Chi phí dự kiến/tháng (USD) | Ghi chú |
| --- | ---: | --- |
| EC2 `t3.small` | 18–22 | Phụ thuộc giá tại Singapore và số giờ chạy |
| EBS và public IPv4 | 4–6 | Root volume và địa chỉ IPv4 công khai |
| Amazon S3 | 0.3–1.5 | 10 GB storage, request và lifecycle |
| Amazon CloudFront | 0–3 | Có thể nằm trong hạn mức miễn phí ở tải nhỏ |
| AWS Certificate Manager | 0 | Public certificate không export dùng với CloudFront |
| Amazon ECR | 0–0.2 | Giữ số image giới hạn bằng lifecycle policy |
| Amazon CloudWatch và SNS | 0–2 | Log, hai standard alarms và email notification |
| MongoDB Atlas M0 | 0 | Dịch vụ ngoài AWS, dùng tier miễn phí |
| Groq | 0 hoặc theo gói | AI provider ngoài AWS đang được ứng dụng sử dụng |
| Tên miền và DNS Tenten | Theo kỳ đăng ký | Dịch vụ ngoài AWS, tên miền đã được đăng ký |
| **Tổng AWS ước tính** | **23–35/tháng** | Chưa tính thuế, Groq, phí tên miền Tenten và tải cao |

Đây là phạm vi dự toán cho môi trường học tập, không phải hóa đơn cố định. Chi phí AWS thực tế phụ thuộc Region, lưu lượng, video và chương trình Free Tier/credit của tài khoản; mức sử dụng Groq được tính riêng theo gói dịch vụ bên ngoài đã chọn. Bảng giá tham khảo gồm [Amazon EC2 On-Demand](https://aws.amazon.com/ec2/pricing/on-demand/), [Amazon S3](https://aws.amazon.com/s3/pricing/) và [Amazon CloudFront](https://aws.amazon.com/cloudfront/pricing/). Trước khi mở rộng cần tạo workload estimate trong [AWS Pricing Calculator](https://calculator.aws/) và thiết lập AWS Budget.

### 9. Đánh giá rủi ro

| Rủi ro | Mức ảnh hưởng | Khả năng | Biện pháp giảm thiểu |
| --- | --- | --- | --- |
| Groq API bị throttling, hết quota hoặc trả response sai cấu trúc | Cao | Trung bình | Rate limit theo user, cache summary, giới hạn prompt, kiểm tra response, timeout, retry và lỗi rõ ràng |
| EC2 đơn gặp sự cố | Cao | Thấp–Trung bình | Health check, StatusCheckFailed alarm, deploy rollback; tương lai dùng ALB và Auto Scaling Multi-AZ |
| Upload video lớn chậm hoặc gián đoạn | Trung bình | Trung bình | Presigned multipart, progress, retry/abort, cleanup session; tương lai dùng media CDN/HLS |
| PDF scan làm OCR tốn RAM/CPU | Trung bình | Cao | Giới hạn trang/kích thước, OCR tuần tự, timeout, concurrency bằng 1 và run ID |
| File rác trên S3 làm tăng chi phí | Trung bình | Trung bình | Safe-delete, cleanup queue, orphan upload cleanup và lifecycle rule |
| Sai DNS, certificate, CORS, cookie hoặc CloudFront behavior | Cao | Trung bình | Giữ CNAME validation ACM, dùng `www` làm canonical origin, same-origin `/api`, allowlist production và tách cache behavior |
| OIDC/IAM cấu hình sai | Cao | Trung bình | Trust policy giới hạn repository, least privilege và không dùng access key dài hạn |
| Secret trong `.env` bị lộ | Cao | Thấp | Quyền file `600`, không commit; tương lai chuyển sang Secrets Manager/Parameter Store |
| MongoDB Atlas hoặc Groq gián đoạn | Trung bình | Thấp–Trung bình | Ready health check, xử lý timeout, log lỗi, backup và phương án provider thay thế |
| Chi phí tăng do AI và storage | Trung bình | Trung bình | Ghi token, AI rate limit, cache kết quả, cleanup S3, AWS Budget và theo dõi CloudWatch |

### 10. Kết quả kỳ vọng

#### 10.1. Kết quả chức năng

* Một nền tảng học tập trực tuyến hoàn chỉnh cho học viên, giảng viên và quản trị viên.
* Quản lý khóa học, bài học, đăng ký, tiến độ, quiz, thông báo và thảo luận tập trung.
* Upload và truy cập media S3 an toàn mà không truyền file lớn qua Backend.
* AI Assistant trả lời theo ngữ cảnh bài học, tóm tắt document và hỗ trợ tạo quiz.
* Giảng viên theo dõi tiến độ và chi tiết kết quả học tập của từng học viên.
* Quản trị viên theo dõi tài khoản, nội dung và các metric hệ thống.

#### 10.2. Kết quả kỹ thuật

* Frontend và Backend được triển khai trên AWS và truy cập qua `https://www.learnsphere.id.vn`.
* CI/CD tự động sử dụng credential tạm thời, health check và rollback an toàn.
* Private storage, least-privilege IAM, secure cookie và request rate limiting.
* Logging, health endpoint, system metrics, CloudWatch alarms và SNS notifications.
* Kiến trúc có thể mở rộng mà không cần thay đổi toàn bộ mã nguồn.

#### 10.3. Giá trị vận hành và chỉ số đánh giá

* Trích xuất document, tái sử dụng bản tóm tắt và tạo bản nháp quiz bằng AI giúp giảm công việc chuẩn bị lặp lại; giảng viên vẫn chịu trách nhiệm kiểm tra nội dung được sinh.
* AI Assistant bám theo nội dung bài học hỗ trợ học viên ngoài giờ lên lớp, trong phạm vi khả dụng của Groq và giới hạn request đã cấu hình.
* Upload trực tiếp lên S3 giúp video dung lượng lớn không phải truyền qua EC2, trong khi CloudFront cải thiện việc phân phối nội dung tĩnh.
* CI/CD tự động giúp quá trình triển khai có thể lặp lại, kiểm tra trạng thái Backend và duy trì phương án rollback khi container mới gặp lỗi.
* Hiệu quả cần được đánh giá bằng số liệu gốc và số liệu sau triển khai như thời gian tạo quiz, thời gian deploy, tỷ lệ request AI thành công, chi phí hạ tầng mỗi tháng và thời gian khôi phục sự cố. Chỉ nên công bố tỷ lệ tiết kiệm hoặc thời gian hoàn vốn sau khi có đủ dữ liệu production.

### 11. Hướng phát triển

* Thêm AWS WAF trước CloudFront.
* Đưa Backend sau Application Load Balancer và Auto Scaling Group tại nhiều Availability Zone.
* Chuyển secret sang AWS Secrets Manager hoặc Systems Manager Parameter Store.
* Dùng CloudFront media distribution, HLS và AWS Elemental MediaConvert cho video.
* Đưa OCR/indexing sang hàng đợi bất đồng bộ bằng SQS và worker riêng.
* Bổ sung SES cho email production, backup tự động và Infrastructure as Code.
* Nghiên cứu Retrieval-Augmented Generation (RAG), tìm kiếm ngữ nghĩa và gợi ý lộ trình học cá nhân hóa.
