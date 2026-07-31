---
title: "Phân tích chi phí"
date: 2026-07-31
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

#### Mục tiêu phân tích

Phần này ước tính chi phí vận hành kiến trúc **LearnSphere As-Built** tại Region Singapore (`ap-southeast-1`). Kiến trúc bao gồm hai EC2 `t3.small` trong Auto Scaling Group, hai NAT Gateway, một Application Load Balancer, Amazon S3, Amazon ECR, Amazon CloudFront, CloudWatch và các dịch vụ quản trị liên quan.



#### Giả định đầu vào

| Nhóm giả định | Giá trị sử dụng |
| --- | --- |
| Region | Asia Pacific (Singapore) – `ap-southeast-1` |
| Thời gian hoạt động | 240 giờ/tháng |
| Backend | 2 EC2 `t3.small`, Linux On-Demand |
| High Availability | 2 Availability Zone, 2 private subnet, 2 NAT Gateway |
| Load balancing | 1 internet-facing Application Load Balancer |
| Lưu trữ EC2 | 2 EBS gp3, 8 GB/volume |
| Dữ liệu qua NAT | Khoảng 10 GB/tháng |
| S3 | Khoảng 11 GB cho frontend và media |
| ECR | Khoảng 1 GB image đang lưu |
| CloudFront | Free plan, lưu lượng workshop dưới hạn mức |
| MongoDB Atlas và Groq | Sử dụng gói miễn phí trong giai đoạn workshop |

#### Bảng dự toán chi phí theo tháng

| Dịch vụ / Tài nguyên | Cách tính cho kịch bản workshop | Chi phí ước tính (USD/tháng) |
| --- | --- | ---: |
| Amazon EC2 | 2 × `t3.small` × 240 giờ × 0,0268 USD | 12,86 |
| Amazon EBS gp3 | 2 volume × 8 GB; dữ liệu lưu liên tục | 1,54 |
| NAT Gateway – thời gian hoạt động | 2 NAT Gateway × 240 giờ × 0,059 USD | 28,32 |
| NAT Gateway – dữ liệu xử lý | 10 GB × 0,059 USD | 0,59 |
| Application Load Balancer | 1 ALB × 240 giờ × 0,0252 USD | 6,05 |
| ALB Capacity Unit | Tải workshop trung bình khoảng 0,25 LCU | 0,48 |
| Public IPv4 | 2 địa chỉ IPv4 của NAT Gateway × 240 giờ | 2,40 |
| Amazon S3 Standard | Khoảng 11 GB frontend và media | 0,28 |
| Amazon S3 Requests | PUT/GET ở mức thử nghiệm | 0,05 |
| Amazon ECR | Khoảng 1 GB Docker image | 0,10 |
| Amazon CloudWatch | Log ingestion, log storage và custom metrics ở mức thấp | 0,75 |
| Amazon CloudFront | Free plan, dưới hạn mức của gói | 0,00 |
| ACM, IAM, SSM Parameter Store và S3 Gateway Endpoint | Không tính phí tài nguyên cơ bản trong phạm vi sử dụng hiện tại | 0,00 |
| **Tạm tính hạ tầng** |  | **53,42** |
| **Dự phòng biến động khoảng 5%** | Lưu lượng, request và log phát sinh ngoài giả định | **2,67** |
| **TỔNG DỰ TOÁN** | Kịch bản workshop vận hành có kiểm soát | **56,09 USD/tháng** |

Chi phí mục tiêu vì vậy nằm trong khoảng **50–60 USD/tháng**. Con số này là dự toán trước thuế và có thể thay đổi theo tỷ giá, thời lượng tài nguyên thực tế, lưu lượng mạng và cập nhật đơn giá của AWS.

#### Phân tích cơ cấu chi phí

**1. NAT Gateway là thành phần chiếm tỷ trọng lớn nhất**

Hai NAT Gateway dự kiến tiêu tốn 28,91 USD, gồm thời gian hoạt động và dữ liệu xử lý, tương đương hơn một nửa tổng chi phí hạ tầng. Kiến trúc sử dụng một NAT Gateway cho mỗi Availability Zone để tránh phụ thuộc chéo AZ và giữ khả năng truy cập outbound cho các EC2 private.

S3 Gateway Endpoint được sử dụng để EC2 truy cập Amazon S3 mà không đưa lưu lượng S3 qua NAT Gateway. Cách này giảm dữ liệu tính phí qua NAT và phù hợp với nguyên tắc Cost Optimization.

**2. Compute và lưu trữ Backend**

Hai EC2 `t3.small` duy trì khả năng chạy Backend ở hai Availability Zone. Auto Scaling Group có cấu hình tối thiểu 2 instance khi hệ thống hoạt động, nhờ đó ALB vẫn có target khỏe mạnh nếu một instance gặp sự cố. EBS gp3 cung cấp root volume độc lập cho từng instance.

Trong giai đoạn workshop, EC2 chỉ chạy trong thời gian học, kiểm thử và demo. Khi kết thúc phiên, Auto Scaling Group được scale về 0 để không tiếp tục phát sinh compute hours.

**3. Application Load Balancer**

ALB là điểm vào HTTPS cho Backend và phân phối request đến Target Group trên port 5000. Chi phí gồm thời gian tồn tại của ALB và LCU sử dụng. Do lưu lượng nghiệm thu thấp, phần LCU nhỏ hơn đáng kể so với phí theo giờ.

Để đạt ngân sách 50–60 USD, ALB phải được xóa sau phiên workshop và tái tạo bằng cấu hình đã ghi lại khi cần. Nếu giữ ALB liên tục 730 giờ/tháng, chi phí sẽ cao hơn bảng dự toán.

**4. Frontend, media và phân phối nội dung**

Frontend tĩnh được lưu trong S3 và phân phối bằng CloudFront. Media được lưu ở bucket riêng và truy cập qua presigned URL. Thiết kế này tránh truyền video qua EC2, giảm tải Backend và giúp chi phí S3 phụ thuộc trực tiếp vào dung lượng cùng số request.

CloudFront đang dùng Free plan và lưu lượng workshop thấp hơn hạn mức. Vì vậy bảng chưa ghi nhận chi phí CloudFront bổ sung. Khi hệ thống có nhiều học viên hoặc vượt hạn mức, cần cập nhật lại estimate.

**5. Logging và dịch vụ quản trị**

CloudWatch tập trung log từ các EC2 trong Auto Scaling Group và theo dõi alarm. IAM, ACM, SSM Parameter Store và S3 Gateway Endpoint không phát sinh phí tài nguyên cơ bản theo cấu hình hiện tại, nhưng KMS key do khách hàng quản lý, advanced parameter hoặc lượng API lớn có thể tạo thêm chi phí.


#### Biện pháp kiểm soát và tối ưu chi phí

1. Duy trì AWS Budget và cảnh báo tại các ngưỡng 50%, 80% và 100%.
2. Kiểm tra Cost Explorer theo từng service, đặc biệt là `EC2-Other`, NAT Gateway và Elastic Load Balancing.
3. Scale Auto Scaling Group về 0 sau khi kết thúc demo.
4. Xóa ALB, Target Group không dùng, NAT Gateway và giải phóng Elastic IP khi tạm dừng môi trường.
5. Dùng S3 Gateway Endpoint cho lưu lượng đến S3.
6. Đặt ECR lifecycle policy để xóa image cũ, chỉ giữ các bản cần rollback.
7. Đặt CloudWatch Logs retention thay vì lưu log vô thời hạn.
8. Theo dõi dung lượng S3 Media và bổ sung lifecycle policy khi có nhiều video cũ.
9. Khi chuyển sang production 24/7, đánh giá Savings Plans cho EC2 và cân nhắc kiến trúc egress có chi phí phù hợp hơn.

#### Kết luận

Với thời gian vận hành được giới hạn và quy trình dọn tài nguyên được thực hiện đúng, LearnSphere có thể duy trì kiến trúc Multi-AZ để phục vụ workshop trong ngân sách khoảng **56,09 USD/tháng**. Dự toán này không che giấu chi phí của NAT Gateway hoặc ALB, không phụ thuộc vào một khoản credit giả định, và tách biệt rõ giữa môi trường workshop với production 24/7.

