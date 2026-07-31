---
title: "Worklog Tuần 3"
date: 2026-06-15
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3

* Hiểu cách AWS cung cấp và quản lý cơ sở dữ liệu quan hệ thông qua Amazon RDS.
* Nắm được các khái niệm cơ bản của cơ sở dữ liệu NoSQL với Amazon DynamoDB.
* Tìm hiểu cách Amazon CloudFront phân phối nội dung và cải thiện tốc độ truy cập ứng dụng.
* Hiểu quy trình đóng gói ứng dụng bằng Docker và lưu trữ Docker image trên Amazon ECR.
* Thực hành kết hợp các dịch vụ AWS để chuẩn bị cho việc triển khai ứng dụng ở những tuần sau.

### Công việc thực hiện trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| **2** | - Tìm hiểu tổng quan về **Amazon Relational Database Service (RDS)**.<br>- Nghiên cứu các thành phần và khái niệm chính:<br>&emsp;+ Database Engine<br>&emsp;+ DB Instance<br>&emsp;+ DB Instance Class<br>&emsp;+ Storage<br>&emsp;+ Security Group<br>&emsp;+ Automated Backup<br>&emsp;+ Multi-AZ Deployment<br>- Tìm hiểu cách ứng dụng kết nối an toàn đến cơ sở dữ liệu RDS trong VPC. | 15/06/2026 | 15/06/2026 | https://000005.awsstudygroup.com/ |
| **3** | - Tìm hiểu mô hình cơ sở dữ liệu NoSQL và dịch vụ **Amazon DynamoDB**.<br>- Nghiên cứu các thành phần cơ bản:<br>&emsp;+ Table và Item<br>&emsp;+ Attribute<br>&emsp;+ Partition Key<br>&emsp;+ Sort Key<br>&emsp;+ Capacity Mode<br>- Thực hành tạo bảng và thực hiện các thao tác thêm, đọc, cập nhật, xóa dữ liệu cơ bản. | 16/06/2026 | 16/06/2026 | https://000060.awsstudygroup.com/ |
| **4** | - Tìm hiểu **Amazon CloudFront** và vai trò của Content Delivery Network (CDN).<br>- Nghiên cứu các khái niệm:<br>&emsp;+ Distribution<br>&emsp;+ Origin<br>&emsp;+ Edge Location<br>&emsp;+ Cache Behavior<br>&emsp;+ Cache Invalidation<br>- Thực hành tạo CloudFront distribution sử dụng S3 bucket làm origin.<br>- Tìm hiểu cách giới hạn quyền truy cập trực tiếp vào S3 và phân phối nội dung qua HTTPS. | 17/06/2026 | 17/06/2026 | https://000094.awsstudygroup.com/ |
| **5** | - Tìm hiểu kiến thức nền tảng về **Docker** và vai trò của container trong quá trình phát triển phần mềm.<br>- Phân biệt Docker Image, Container và Registry.<br>- Tìm hiểu cấu trúc Dockerfile và các chỉ thị cơ bản:<br>&emsp;+ FROM<br>&emsp;+ WORKDIR<br>&emsp;+ COPY<br>&emsp;+ RUN<br>&emsp;+ EXPOSE<br>&emsp;+ CMD<br>- Thực hành tạo Dockerfile, build image và chạy một ứng dụng mẫu trong container. | 18/06/2026 | 18/06/2026 | https://000015.awsstudygroup.com/ |
| **6** | - Tìm hiểu **Amazon Elastic Container Registry (ECR)** và quy trình quản lý Docker image.<br>- Thực hành:<br>&emsp;+ Tạo ECR repository<br>&emsp;+ Xác thực Docker với ECR<br>&emsp;+ Gắn tag cho Docker image<br>&emsp;+ Push image lên ECR<br>&emsp;+ Pull image từ ECR<br>- Chạy thử container từ image lưu trên ECR và tổng hợp quy trình đóng gói, lưu trữ, phân phối ứng dụng. | 19/06/2026 | 19/06/2026 | https://000017.awsstudygroup.com/ |

### Kết quả đạt được tuần 3

* Hiểu vai trò của Amazon RDS và các thành phần cơ bản:
  * Database Engine
  * DB Instance
  * Storage
  * Security Group
  * Automated Backup
  * Multi-AZ Deployment

* Nắm được các khái niệm Table, Item, Attribute, Partition Key và Sort Key trong Amazon DynamoDB.
* Hiểu cách Amazon CloudFront sử dụng hệ thống Edge Location và bộ nhớ đệm để phân phối nội dung.
* Tạo được CloudFront distribution sử dụng Amazon S3 làm origin.
* Hiểu sự khác nhau giữa Docker Image, Container và Registry.
* Tạo Dockerfile, build Docker image và chạy ứng dụng mẫu trong container.
* Tạo Amazon ECR repository và thực hiện được quy trình xác thực, gắn tag, push, pull Docker image.
* Xây dựng được nền tảng về cơ sở dữ liệu, phân phối nội dung và container để chuẩn bị triển khai ứng dụng trên AWS.
