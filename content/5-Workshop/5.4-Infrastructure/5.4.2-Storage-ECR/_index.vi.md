---
title: "Amazon S3 và Amazon ECR"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

#### Mục tiêu

LearnSphere tách ba loại artifact thành ba tài nguyên độc lập:

| Tài nguyên | Dữ liệu | Cách truy cập |
| --- | --- | --- |
| `learnsphere-fe-2` | Bản build React/Vite | CloudFront Origin Access Control |
| `learnsphere-media-2` | Video, document, thumbnail và avatar | Presigned URL do Backend tạo |
| `learnsphere-be-2` | Docker image Backend | IAM role của GitHub Actions và EC2 |

Việc tách này giúp mỗi loại dữ liệu có policy, vòng đời và quy trình triển khai riêng.

#### 1. Tạo S3 Frontend

Tạo bucket `learnsphere-fe-2` tại `ap-southeast-1` với các thiết lập:

* Object Ownership: Bucket owner enforced.
* Block Public Access: bật toàn bộ.
* Static website hosting: không bắt buộc vì CloudFront dùng REST origin.
* Versioning: có thể bật để hỗ trợ khôi phục object.
* Encryption: SSE-S3 hoặc SSE-KMS theo yêu cầu.

CloudFront đọc bucket thông qua Origin Access Control. Bucket policy chỉ cho CloudFront service principal đọc object khi `AWS:SourceArn` khớp distribution production.

Frontend được triển khai bằng:

```bash
aws s3 sync LearnSphere_FE/dist/ s3://learnsphere-fe-2 \
  --delete \
  --region ap-southeast-1
```

Sau đồng bộ, pipeline tạo CloudFront invalidation để người dùng nhận phiên bản mới.

#### 2. Tạo S3 Media

Tạo bucket `learnsphere-media-2` tại cùng Region và giữ bucket private. Backend lưu object theo prefix nghiệp vụ, ví dụ:

```text
courses/<courseId>/thumbnail/...
courses/<courseId>/lessons/<lessonId>/videos/...
courses/<courseId>/lessons/<lessonId>/documents/...
users/<userId>/avatars/...
```

Trình duyệt không có AWS credential. Backend xác thực người dùng, kiểm tra role và sinh presigned URL có thời hạn. Video lớn sử dụng multipart upload; session hết hạn và object mồ côi được tác vụ cleanup xử lý.

#### 3. Cấu hình CORS cho Media

Các origin được phép phải khớp chính xác protocol và domain:

```json
[
  {
    "AllowedOrigins": [
      "http://localhost:5173",
      "http://127.0.0.1:5173",
      "https://dzr6s0675pe82.cloudfront.net",
      "https://www.learnspherev2.id.vn"
    ],
    "AllowedMethods": ["GET", "PUT", "POST", "HEAD"],
    "AllowedHeaders": ["*"],
    "ExposeHeaders": ["ETag"]
  }
]
```

`ETag` được expose để Frontend hoàn tất multipart upload. CORS không thay thế IAM hoặc bucket policy; nó chỉ kiểm soát request của trình duyệt. Bucket vẫn chặn public access.

#### 4. Cấu hình bảo vệ và vòng đời

Các thiết lập nên kiểm tra:

* Default encryption được bật trên cả hai bucket.
* Không có ACL public.
* Incomplete multipart uploads được abort sau thời hạn phù hợp.
* Có lifecycle rule cho object tạm hoặc phiên bản cũ nếu project sử dụng versioning.
* S3 Media không cấp `s3:ListBucket` hoặc quyền quản lý object cho người dùng cuối.
* Backend xóa object cũ sau khi database đã cập nhật thành công; file mới được cleanup nếu cập nhật database thất bại.

#### 5. Tạo ECR repository

Tạo private repository `learnsphere-be-2` tại `ap-southeast-1`. Docker image được tag bằng toàn bộ Git commit SHA:

```text
440893644584.dkr.ecr.ap-southeast-1.amazonaws.com/learnsphere-be-2:<git-sha>
```

Tag theo commit SHA tạo quan hệ một-một giữa source code và artifact triển khai. Pipeline kiểm tra image đã tồn tại trước khi build, sau đó push image bất biến lên ECR.

Ví dụ:

```bash
aws ecr get-login-password --region ap-southeast-1 |
docker login \
  --username AWS \
  --password-stdin 440893644584.dkr.ecr.ap-southeast-1.amazonaws.com

docker build -t learnsphere-be:<git-sha> LearnSphere_BE
docker tag learnsphere-be:<git-sha> \
  440893644584.dkr.ecr.ap-southeast-1.amazonaws.com/learnsphere-be-2:<git-sha>
docker push \
  440893644584.dkr.ecr.ap-southeast-1.amazonaws.com/learnsphere-be-2:<git-sha>
```

ECR lifecycle policy có thể giữ một số image gần nhất và xóa image không còn tag để hạn chế chi phí. Image đang được `/learnsphere/prod/backend-image-tag` tham chiếu không được xóa.

#### 6. Xác minh

* Mở từng S3 bucket và xác nhận **Block Public Access = On**.
* Kiểm tra bucket Frontend có các file `index.html` và `assets/` sau pipeline.
* Kiểm tra Media CORS có domain production.
* Mở ECR và xác nhận image tag là commit SHA, trạng thái scan và thời gian push.
* Thử tải trực tiếp S3 object không có presigned URL; request phải bị từ chối.

#### Kết quả triển khai lưu trữ và container registry

![Hai S3 bucket production của LearnSphere](/images/learnsphere-s3-buckets.png)

*Hình 5.11. Hai bucket `learnsphere-fe-2` và `learnsphere-media-2` được tạo tại AWS Region Singapore (`ap-southeast-1`) để tách riêng bản build Frontend và học liệu của hệ thống.*

![Danh sách Docker image trong Amazon ECR](/images/learnsphere-ecr-images.png)

*Hình 5.12. Repository `learnsphere-be-2` lưu các Docker image Backend với tag là Git commit SHA, cho phép truy vết chính xác source code tương ứng với từng phiên bản đã phát hành.*

Kết quả cho thấy static artifact, media object và Backend container image đã được phân tách thành các vùng lưu trữ độc lập. Cấu trúc này giúp CI/CD chỉ cập nhật đúng tài nguyên cần thiết và hỗ trợ rollback Backend bằng cách chọn lại một commit SHA đã tồn tại trong ECR.
