---
title: "S3 Media và luồng upload an toàn"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.8.2. </b> "
---

#### 1. Tách control plane và data plane

Backend xác thực quyền và cấp URL, nhưng binary đi trực tiếp giữa trình duyệt và S3:

```text
Browser → POST /api/files/presigned-upload → Backend
Browser ← upload URL + session ID           ← Backend
Browser → PUT binary                         → S3 Media
Browser → confirm session                    → Backend
Backend → HEAD object                        → S3 Media
Backend → lưu file key                       → MongoDB
```

Thiết kế này tránh giữ video 100–200 MB trong memory/container và giảm băng thông đi qua ALB/EC2.

#### 2. Single-part upload

Thumbnail, avatar và document nhỏ dùng:

```text
POST /api/files/presigned-upload
POST /api/files/profile-avatar/presigned-upload
POST /api/files/uploads/:session_id/confirm
```

Backend kiểm tra role, course ownership, folder, MIME type, kích thước và file name trước khi ký URL. Khi confirm, Backend dùng S3 `HeadObject` để so khớp metadata trước khi đánh dấu session hoàn tất.

#### 3. Multipart video upload

Frontend chọn multipart khi video từ 25 MB:

```text
POST /api/files/multipart/start
→ upload từng part bằng presigned URL
→ thu thập ETag
POST /api/files/multipart/:session_id/complete
```

Client upload tối đa ba part đồng thời và hiển thị phần trăm. Nếu một part thất bại, client gọi:

```text
DELETE /api/files/uploads/:session_id
```

để hủy session thay vì để multipart upload treo.

#### 4. Download và quyền truy cập

File private được tải bằng:

```text
GET /api/files/presigned-download
GET /api/files/course-thumbnail/:course_id
GET /api/files/profile-avatar
```

Backend kiểm tra enrollment/role trước khi cấp URL có thời hạn cho file riêng tư; thumbnail khóa học được cấp qua endpoint public riêng để hiển thị catalog. Bucket Media vẫn không trở thành public file server.

#### 5. Dọn file cũ và orphan

Hệ thống theo dõi `UploadSession` và chạy cleanup định kỳ:

* Abort multipart session hết hạn.
* Xóa object đã upload nhưng không được gắn vào dữ liệu.
* Xóa file cũ sau khi DB đã lưu key mới.
* Xóa prefix khóa học khi xóa vĩnh viễn.
* Giữ file khi course chỉ soft-delete để còn restore.

Thứ tự “lưu key mới thành công rồi mới xóa file cũ” tránh làm mất dữ liệu đang dùng nếu S3 hoặc database gặp lỗi giữa chừng.

#### Bằng chứng triển khai

![Các prefix cấp cao nhất trong bucket media riêng tư của LearnSphere](/images/learnsphere-s3-media-prefixes.png)

**Hình 5.49:** Bucket riêng tư `learnsphere-media-2` tách học liệu khóa học trong `courses/` và tài nguyên người dùng trong `users/`. Ảnh không làm lộ presigned URL hoặc nội dung của object riêng tư.

![Tiến độ multipart upload video trên giao diện quản lý bài học](/images/learnsphere-multipart-video-upload.png)

**Hình 5.50:** Giao diện quản lý bài học hiển thị tiến độ của video dung lượng lớn được tải theo từng phần. Tutor nhận được phản hồi trực quan trong thời gian trình duyệt hoàn tất quá trình truyền file thay vì phải chờ một request không có trạng thái.
