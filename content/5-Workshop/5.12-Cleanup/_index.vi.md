---
title: "Dọn dẹp tài nguyên"
date: 2026-07-30
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

Cleanup là giai đoạn ngừng vận hành có kiểm soát sau khi dự án đã được nghiệm thu. Mục tiêu không chỉ là xóa tài nguyên mà còn phải bảo toàn dữ liệu cần lưu, tránh làm gián đoạn tài nguyên dùng chung và xác nhận không còn dịch vụ tiếp tục phát sinh chi phí.


#### 1. Xác định phạm vi dọn dẹp

Trước khi xóa, lập danh sách tài nguyên thuộc LearnSphere theo Region và theo loại dịch vụ. Các tài nguyên chính của hệ thống gồm:

* CloudFront distribution, CloudFront Function và Origin Access Control.
* Hai S3 bucket `learnsphere-fe-2` và `learnsphere-media-2`.
* Application Load Balancer, HTTPS listener và Backend Target Group.
* Auto Scaling Group, Launch Template và các EC2 instance do ASG quản lý.
* ECR repository `learnsphere-be-2` cùng các Docker image.
* VPC production, bốn subnet, Internet Gateway, hai NAT Gateway, hai Elastic IP, route table và S3 Gateway Endpoint.
* Parameter Store, CloudWatch log group, alarm và SNS topic.
* IAM role cho GitHub OIDC, EC2 instance profile và các policy liên quan.
* Chứng chỉ ACM tại `us-east-1` cho CloudFront và tại `ap-southeast-1` cho ALB.
* DNS record tại nhà cung cấp tên miền, MongoDB Atlas và Groq API key.

Đối chiếu tên, tag `Project=LearnSphere`, ARN và Region trước mỗi thao tác. Không xóa service-linked role, OIDC provider hoặc tài nguyên dùng chung nếu chúng còn được dự án khác sử dụng.

#### 2. Sao lưu dữ liệu và bằng chứng

Thực hiện sao lưu trước khi thay đổi lưu lượng hoặc hạ tầng:

1. Export các collection cần giữ từ MongoDB Atlas, đồng thời ghi lại database, cluster và chính sách IP Access List.
2. Sao lưu các object quan trọng trong S3 Media, đặc biệt là video, tài liệu, thumbnail và avatar.
3. Lưu source code, Dockerfile, workflow GitHub Actions và commit cuối cùng đã chạy ổn định.
4. Ghi lại cấu hình VPC, route table, Security Group, ALB, Target Group, ASG, Launch Template và CloudFront.
5. Lưu tên Parameter Store và cấu trúc biến môi trường nhưng không đưa giá trị SecureString vào báo cáo hoặc Git.
6. Xuất các log, chỉ số, kết quả kiểm thử và thông tin chi phí cần dùng làm bằng chứng workshop.
7. Kiểm tra bản sao lưu có thể mở hoặc import được trước khi bắt đầu xóa.

#### 3. Ngăn phát sinh bản triển khai mới

Trước tiên phải ngăn pipeline tạo lại tài nguyên hoặc khởi động một đợt Instance Refresh mới:

1. Tạm dừng trigger `push` của workflow hoặc vô hiệu hóa GitHub Actions cho repository.
2. Xác nhận không còn workflow run, ASG Instance Refresh hoặc scaling activity đang thực thi.
3. Không xóa GitHub Secrets và IAM role ngay ở bước này vì có thể vẫn cần quyền để kiểm tra hoặc hoàn tất cleanup.
4. Ghi lại image tag production cuối cùng trong `/learnsphere/prod/backend-image-tag`.

#### 4. Dừng Backend High Availability

Backend phải được dừng từ lớp điều phối trước, không terminate thủ công từng EC2 vì ASG sẽ tự tạo instance thay thế.

1. Tắt scheduled action và scaling policy nếu có.
2. Chỉnh ASG về `min=0`, `desired=0`; có thể giữ `max=4` trong lúc chờ.
3. Theo dõi tab **Instance management** đến khi không còn EC2 instance `InService` hoặc `Terminating`.
4. Kiểm tra Target Group không còn target được đăng ký.
5. Xóa Auto Scaling Group sau khi capacity đã về 0.
6. Xóa toàn bộ phiên bản Launch Template chỉ dành cho LearnSphere.
7. Kiểm tra EC2 console và network interface để chắc chắn không còn instance hoặc ENI do Backend tạo ra.

Sau khi Backend dừng, endpoint `/health/ready` và các API sẽ không còn hoạt động. Đây là trạng thái dự kiến trong quá trình decommission.

#### 5. Xóa lớp cân bằng tải

Thực hiện sau khi ASG đã ngừng để tránh target tiếp tục đăng ký vào Target Group:

1. Xóa HTTPS listener và các listener rule của ALB.
2. Xóa `LearnSphere-Prod-ALB`.
3. Chờ ALB được xóa hoàn toàn và các ENI liên quan biến mất.
4. Xóa `LearnSphere-Backend-TG`.
5. Chỉ xóa `LearnSphere-ALB-SG` và `LearnSphere-Backend-SG` khi không còn tài nguyên tham chiếu.

Nếu AWS báo tài nguyên vẫn đang được sử dụng, kiểm tra liên kết ASG–Target Group, listener, ENI và Security Group thay vì cố xóa theo thứ tự ngược.

#### 6. Dọn mạng VPC theo dependency

NAT Gateway là tài nguyên cần ưu tiên vì tiếp tục tính phí theo giờ và dữ liệu xử lý cho đến khi bị xóa.

1. Xóa hai NAT Gateway thuộc public subnet 1a và 1b.
2. Chờ trạng thái chuyển sang `Deleted`, sau đó release hai Elastic IP tương ứng.
3. Xóa S3 Gateway Endpoint nếu không còn workload sử dụng.
4. Xóa route `0.0.0.0/0` đến NAT Gateway trong từng private route table.
5. Xóa các route table tùy chỉnh sau khi bỏ liên kết subnet.
6. Xóa hai private subnet và hai public subnet.
7. Xóa các Security Group tùy chỉnh và Network ACL tùy chỉnh còn lại.
8. Detach Internet Gateway khỏi VPC rồi mới xóa Internet Gateway.
9. Xóa VPC production sau khi không còn subnet, ENI, endpoint, gateway hoặc tài nguyên phụ thuộc.

Nếu mục tiêu chỉ là tạm dừng hệ thống thay vì xóa vĩnh viễn, có thể giữ VPC và subnet nhưng vẫn nên xóa NAT Gateway để tránh chi phí cố định.

#### 7. Dọn Frontend, CloudFront và S3

CloudFront và S3 cần được xử lý cẩn thận vì distribution vẫn tham chiếu đến origin bucket:

1. Gỡ custom domain khỏi CloudFront nếu tên miền sẽ được chuyển sang hệ thống khác.
2. Disable CloudFront distribution và chờ trạng thái triển khai hoàn tất.
3. Xóa distribution, CloudFront Function và Origin Access Control chỉ dành cho LearnSphere.
4. Với S3 Frontend, xóa `index.html`, thư mục `assets/` và các object còn lại.
5. Với S3 Media, chỉ xóa sau khi đã sao lưu dữ liệu cần giữ.
6. Nếu bucket bật versioning, phải xóa cả object version và delete marker.
7. Hủy các multipart upload chưa hoàn tất để không tiếp tục chiếm dung lượng.
8. Xóa bucket policy, CORS rule và cuối cùng xóa hai bucket.

Việc xóa S3 Media trước MongoDB có thể làm các key trong database trỏ đến object không còn tồn tại. Vì vậy phải xác nhận hệ thống đã dừng và dữ liệu đã được sao lưu.

#### 8. Dọn image, cấu hình và giám sát

Sau khi không còn EC2 cần khởi động từ image production:

1. Xóa các image trong ECR rồi xóa repository `learnsphere-be-2`.
2. Xóa `/learnsphere/prod/backend-image-tag` và SecureString `/learnsphere/prod/backend-env`.
3. Xóa CloudWatch alarm, log stream và log group dành riêng cho Backend.
4. Kiểm tra retention hoặc export log trước khi xóa nếu cần lưu lịch sử vận hành.
5. Xóa SNS subscription, sau đó xóa topic `LearnSphere-Alerts2`.
6. Kiểm tra CloudWatch dashboard và metric filter tùy chỉnh còn tham chiếu đến tài nguyên cũ.

Parameter Store phải được xóa sau Backend để tránh làm hỏng bootstrap trong khi ASG vẫn còn khả năng tạo instance mới.

#### 9. Thu hồi IAM, chứng chỉ và DNS

Thực hiện phần bảo mật cuối cùng sau khi các workload đã dừng:

1. Xóa inline policy và managed policy chỉ dành cho deployment LearnSphere.
2. Xóa GitHub deploy role và EC2 role/instance profile khi không còn tài nguyên sử dụng.
3. Chỉ xóa GitHub OIDC provider nếu không repository hoặc dự án nào khác dùng provider này.
4. Xóa các GitHub Secrets như role ARN, bucket name, CloudFront distribution ID và API base URL cũ.
5. Xóa chứng chỉ ACM cho `www.learnspherev2.id.vn` tại `us-east-1` sau khi CloudFront đã bị xóa.
6. Xóa chứng chỉ ACM cho `origin.learnspherev2.id.vn` tại `ap-southeast-1` sau khi ALB đã bị xóa.
7. Xóa record `www`, `origin` và record xác thực ACM không còn cần thiết tại TenTen.
8. Giữ tên miền nếu còn sử dụng cho portfolio hoặc báo cáo; nếu không, tắt tự động gia hạn theo nhu cầu.

#### 10. Xử lý MongoDB Atlas và Groq

MongoDB Atlas và Groq nằm ngoài AWS nên phải được kiểm tra riêng:

* Gỡ hai Elastic IP của NAT Gateway khỏi MongoDB Atlas IP Access List.
* Pause hoặc xóa MongoDB cluster sau khi export dữ liệu; xóa project chỉ khi không chứa database khác.
* Thu hồi hoặc xoay vòng Groq API key đã dùng cho production.
* Xóa biến môi trường, secret cục bộ và file `.env` khỏi máy không còn sử dụng.
* Không đưa database URI, API key hoặc thông tin đăng nhập vào tài liệu cleanup.

#### 11. Xác nhận sau cleanup

Hoàn tất bằng một vòng kiểm tra độc lập:

* Tìm kiếm `LearnSphere` bằng Resource Groups, Tag Editor và từng console dịch vụ tại `ap-southeast-1`.
* Kiểm tra thêm các dịch vụ global hoặc Region khác, đặc biệt là CloudFront, IAM và ACM tại `us-east-1`.
* Xác nhận EC2, ASG, ALB, NAT Gateway, Elastic IP, ECR và S3 không còn tài nguyên ngoài ý muốn.
* Xác nhận DNS không còn trỏ đến CloudFront hoặc ALB đã bị xóa.
* Kiểm tra GitHub Actions không còn khả năng deploy bằng role cũ.
* Theo dõi Billing và Cost Explorer trong 24–48 giờ vì dữ liệu chi phí có độ trễ.
* Lưu biên bản gồm thời gian cleanup, người thực hiện, tài nguyên được giữ lại và tài nguyên đã xóa.

Khi các bước trên hoàn tất, LearnSphere được xem là đã decommission an toàn: dữ liệu cần thiết đã được bảo toàn, credential đã được thu hồi và không còn hạ tầng production bị bỏ quên tiếp tục phát sinh chi phí.
