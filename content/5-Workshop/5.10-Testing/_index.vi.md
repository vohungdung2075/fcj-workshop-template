---
title: "Kiểm thử và kết quả"
date: 2026-07-30
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

#### Phạm vi và điều kiện bắt đầu kiểm thử

Nghiệm thu production chỉ bắt đầu sau khi GitHub Actions của commit mới nhất thành công, CloudFront hoàn tất triển khai, ALB có hai target Healthy và nhóm kiểm thử có tài khoản cho từng vai trò. Dữ liệu dùng trong kiểm thử là học liệu mẫu, không sử dụng tài liệu cá nhân hoặc bí mật.

Chiến lược kiểm thử kết hợp:

| Cấp kiểm thử | Mục tiêu chính |
| --- | --- |
| Automated Backend test | Kiểm tra security header, CORS, readiness, environment validation và AI quiz parser |
| Build validation | Xác nhận React/Vite production bundle biên dịch thành công |
| Infrastructure smoke test | Kiểm tra DNS, TLS, CloudFront, ALB, Target Group, ASG và MongoDB readiness |
| Nghiệm thu theo vai trò | Kiểm tra đầy đủ luồng Học viên, Giảng viên và Admin |
| Failure-path test | Xác minh authorization, upload lỗi, AI error và khả năng phục hồi HA |

#### Ma trận nghiệm thu production

| Hạng mục | Cách kiểm tra | Kết quả mong đợi |
| --- | --- | --- |
| DNS/TLS | Mở `https://www.learnspherev2.id.vn` | Chứng chỉ hợp lệ |
| Frontend | Refresh trực tiếp một SPA route | Không trả 404 |
| API readiness | Gọi `https://origin.learnspherev2.id.vn/health/ready` | HTTP 200 và `database: connected` |
| ALB | Target Health | Hai target Healthy |
| Multi-AZ | Kiểm tra ASG instances | Mỗi AZ có ít nhất một instance |
| MongoDB | Health response | `database: connected` |
| Media | Upload/xem thumbnail, document, video | Presigned URL hoạt động |
| AI | Tóm tắt document và sinh quiz | Groq trả kết quả hợp lệ |
| CI/CD | Push commit lên `main` | Hai job Success |

Mỗi test record cần ghi commit, thời điểm, vai trò thực hiện, kết quả mong đợi, kết quả thực tế và bằng chứng. Test chỉ được đánh dấu `Pass` khi cả nghiệp vụ và ranh giới bảo mật đều đúng.

#### Kiểm thử end-to-end theo vai trò

**Học viên**

1. Đăng ký/đăng nhập.
2. Gửi yêu cầu hoặc tham gia khóa học mở.
3. Xem bài học, video và document.
4. Dùng AI Assistant theo nội dung bài học.
5. Làm quiz và xem kết quả.
6. Hủy đăng ký khóa học.

**Giảng viên**

1. Tạo khóa học và bài học.
2. Upload thumbnail, video, document.
3. Duyệt/xóa học viên và xem tiến độ, chi tiết bài làm.
4. Tạo quiz thủ công hoặc bằng AI theo độ khó.
5. Soft-delete và khôi phục khóa học.

**Admin**

1. Quản lý tài khoản.
2. Xem khóa học/quiz theo giảng viên mà không sửa nội dung.
3. Xem System Monitoring.
4. Kiểm tra thông báo và trạng thái hệ thống.

Kiểm thử vai trò đồng thời bao gồm negative authorization: học viên không truy cập trang quản lý của tutor/admin, tutor không quản trị account và Admin chỉ xem dữ liệu course/quiz của tutor mà không âm thầm trở thành tác giả nội dung.

#### Kiểm thử các luồng lỗi

| Tình huống | Kết quả mong đợi |
| --- | --- |
| Request không có xác thực | Trả `401`, không lộ dữ liệu bảo vệ |
| Đã đăng nhập nhưng sai role | Trả `403` |
| Upload sai loại hoặc quá dung lượng | Lỗi validation rõ ràng, không để lại orphan object |
| Presigned URL hết hạn | Từ chối truyền file và yêu cầu URL mới |
| Groq hết quota hoặc timeout | Trả mã lỗi AI đã chuẩn hóa, trang không dùng AI vẫn hoạt động |
| AI trả cấu trúc quiz sai | Từ chối draft, không lưu câu hỏi lỗi |
| MongoDB mất kết nối | Readiness thất bại và target rời khỏi ALB rotation |

#### Xác minh High Availability

1. Xác nhận Target Group có hai target Healthy.
2. Chọn một instance ASG và terminate thủ công.
3. Trong thời gian thay thế, gọi health endpoint và thao tác website.
4. Xác minh ALB tiếp tục phục vụ qua target còn lại.
5. Chờ ASG tạo instance mới bằng Launch Template mặc định.
6. Xác nhận Target Group trở về hai target Healthy, phân bố qua hai AZ.

Bài kiểm thử terminate phải thực hiện trong thời gian có giám sát. `min` và `desired` vẫn giữ ở mức `2`; nếu giảm các giá trị này thì bài kiểm thử đã thay đổi chính yêu cầu HA cần xác minh. Kết quả đạt khi health endpoint liên tục hoạt động qua target còn lại và hệ thống tự trở về hai target Healthy mà không cài đặt thủ công.

#### Kết quả ghi nhận

Health endpoint production đã trả:

```http
HTTP/1.1 200 OK
```

```json
{"status":"ready","database":"connected"}
```

Backend automated suite hoàn tất 15 test không có lỗi và Frontend production build thành công. GitHub Actions hoàn tất cả Backend lẫn Frontend deployment job. ASG duy trì hai instance tại hai Availability Zone, Target Group báo cáo hai target Healthy.

#### Tiêu chí kết thúc

Production được chấp nhận khi không còn lỗi Critical/High chưa xử lý; DNS, TLS, API và nested SPA route đều pass; hai target Healthy; ba luồng vai trò hoàn tất; media và AI hoạt động; release truy vết được về commit thành công. Bất kỳ lỗi bảo mật, mất dữ liệu hoặc HA nào cũng chặn phát hành.

#### Bằng chứng nghiệm thu

![API production trả về HTTP 200](/images/learnsphere-production-api-patch-200.png)

*Hình 5.58. DevTools ghi nhận request `PATCH /api/users/me` trên domain production `https://www.learnspherev2.id.vn` trả về `HTTP 200 OK`. Kết quả chứng minh CloudFront đã chuyển tiếp đúng luồng `/api/*` đến ALB và Backend xử lý thành công một request có xác thực.*

![Lịch sử thay thế EC2 của Auto Scaling Group](/images/learnsphere-asg-instance-refresh-activity.png)

*Hình 5.59. Activity history của `LearnSphere-Backend-ASG` ghi nhận các thao tác launch instance mới và terminate instance cũ đều ở trạng thái `Successful` trong quá trình Instance Refresh. Kết hợp với bằng chứng Target Group có hai target Healthy ở Hình 5.8, kết quả xác nhận Backend có thể triển khai cuốn chiếu và tự phục hồi mà không cài đặt thủ công từng EC2.*
