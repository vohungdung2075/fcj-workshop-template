---
title: "Worklog Tuần 2"
date: 2026-06-08
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2

* Hiểu cách AWS Identity and Access Management (IAM) quản lý danh tính và quyền truy cập tài nguyên.
* Biết cách theo dõi chi phí, tạo cảnh báo ngân sách và ước tính chi phí workload trên AWS.
* Nắm được các thành phần mạng cơ bản trong Amazon VPC và cách chúng kết nối tài nguyên AWS.
* Thực hành khởi tạo, cấu hình và truy cập máy chủ Amazon EC2.
* Tìm hiểu cách lưu trữ và bảo vệ dữ liệu với Amazon S3.

### Công việc thực hiện trong tuần

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| **2** | - Tìm hiểu tổng quan về **AWS Identity and Access Management (IAM)**.<br>- Phân biệt các thành phần:<br>&emsp;+ IAM User<br>&emsp;+ User Group<br>&emsp;+ IAM Role<br>&emsp;+ Policy<br>- Tìm hiểu nguyên tắc cấp quyền tối thiểu và vai trò của xác thực đa yếu tố (MFA) trong bảo vệ tài khoản AWS. | 08/06/2026 | 08/06/2026 | https://000002.awsstudygroup.com/ |
| **3** | - Tìm hiểu **AWS Budgets** trong Billing and Cost Management.<br>- Tìm hiểu cách sử dụng **AWS Pricing Calculator** để dự toán chi phí dịch vụ.<br>- Thực hành:<br>&emsp;+ Tạo Budget và cấu hình cảnh báo chi phí<br>&emsp;+ Tạo Dashboard theo dõi chi phí<br>&emsp;+ Tạo bản ước tính chi phí cho một workload mẫu | 09/06/2026 | 09/06/2026 | https://000007.awsstudygroup.com/ |
| **4** | - Tìm hiểu tổng quan về **Amazon Virtual Private Cloud (VPC)**.<br>- Nghiên cứu các thành phần mạng cơ bản:<br>&emsp;+ CIDR block<br>&emsp;+ Public Subnet và Private Subnet<br>&emsp;+ Route Table<br>&emsp;+ Internet Gateway<br>&emsp;+ Network ACL và Security Group<br>- Phác thảo mô hình VPC cơ bản cho một ứng dụng web. | 10/06/2026 | 10/06/2026 | https://000003.awsstudygroup.com/ |
| **5** | - Thực hành khởi tạo một **Amazon EC2 instance** sử dụng Amazon Linux.<br>- Tạo Key Pair, cấu hình Security Group và kết nối đến EC2 bằng SSH.<br>- Theo dõi trạng thái instance và tìm hiểu các thao tác Start, Stop, Reboot, Terminate.<br>- Tìm hiểu **Amazon Elastic Block Store (EBS)** và mối quan hệ giữa EBS với EC2.<br>- Thực hành tạo, gắn và kiểm tra EBS volume trên EC2. | 11/06/2026 | 11/06/2026 | https://000004.awsstudygroup.com/ |
| **6** | - Tìm hiểu các khái niệm cơ bản của **Amazon S3**:<br>&emsp;+ Bucket và Object<br>&emsp;+ Object Key<br>&emsp;+ Storage Class<br>&emsp;+ Versioning<br>&emsp;+ Block Public Access<br>- Thực hành tạo bucket, tải lên, tải xuống và sắp xếp object.<br>- Kiểm tra quyền truy cập và tìm hiểu cách bảo vệ dữ liệu S3 khỏi truy cập công khai ngoài ý muốn. | 12/06/2026 | 12/06/2026 | https://000057.awsstudygroup.com/ |

### Kết quả đạt được tuần 2

* Hiểu mục đích của IAM và phân biệt được:
  * IAM User
  * User Group
  * IAM Role
  * Policy

* Hiểu nguyên tắc cấp quyền tối thiểu và vai trò của MFA trong bảo vệ tài khoản AWS.
* Biết cách sử dụng AWS Budgets để tạo ngân sách và cấu hình cảnh báo chi phí.
* Biết cách sử dụng AWS Pricing Calculator để tạo bản ước tính chi phí cho một workload.
* Tạo được Dashboard phục vụ việc theo dõi chi phí AWS.
* Nắm được các thành phần mạng cơ bản của Amazon VPC:
  * CIDR block
  * Public Subnet
  * Private Subnet
  * Route Table
  * Internet Gateway
  * Network ACL
  * Security Group
* Khởi tạo và cấu hình thành công một EC2 instance.
* Kết nối đến máy chủ EC2 bằng SSH và thực hiện được các thao tác quản trị cơ bản.
* Hiểu cách EBS cung cấp bộ nhớ khối cho EC2 và thực hành gắn EBS volume vào instance.
* Nắm được các khái niệm Bucket, Object, Storage Class, Versioning và Block Public Access của Amazon S3.
* Thực hiện được các thao tác tạo bucket, tải lên, tải xuống và quản lý object trên Amazon S3.
* Xây dựng được nền tảng về định danh, mạng, máy chủ và lưu trữ để tiếp tục tìm hiểu kiến trúc hệ thống trên AWS.
