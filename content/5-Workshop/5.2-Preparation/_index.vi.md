---
title: "Chuẩn bị triển khai"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Điều kiện cần

| Hạng mục | Yêu cầu |
| --- | --- |
| AWS Account | Có quyền tạo IAM, VPC, EC2, ECR, S3, ALB, ASG, ACM, CloudFront, SSM, CloudWatch và SNS |
| Region | `ap-southeast-1`; riêng certificate CloudFront phải tạo ở `us-east-1` |
| Local tools | Git, Node.js 24, npm, Docker Engine và AWS CLI v2 |
| Source | Clone repository LearnSphere và checkout branch `main` |
| Database | MongoDB Atlas cluster và database user |
| AI | Groq API key và model production |
| Domain | `learnspherev2.id.vn` và quyền quản lý DNS |

#### Quy trình chuẩn bị

Quá trình chuẩn bị bắt đầu bằng việc kiểm tra AWS Account, quyền IAM và Region sẽ sử dụng. Tiếp theo, người triển khai tạo AWS Budget để kiểm soát chi phí, clone mã nguồn từ nhánh `main` và kiểm tra Backend, Frontend cùng Docker trên môi trường local. Sau khi source hoạt động ổn định, cần chuẩn bị MongoDB Atlas, Groq API, tên miền và các biến môi trường production. Cuối cùng, thống nhất quy ước đặt tên, tag tài nguyên và hoàn thành checklist bảo mật trước khi bắt đầu xây dựng hạ tầng AWS.

Các bước nên được thực hiện đúng thứ tự để tránh trường hợp tài nguyên đã phát sinh chi phí nhưng source, certificate, database hoặc secret vẫn chưa sẵn sàng.

#### Tài khoản và quyền truy cập

1. Đăng nhập AWS Account `440893644584` và bật MFA cho root user.
2. Không sử dụng root user cho thao tác hằng ngày.
3. Chọn Region Singapore (`ap-southeast-1`) trước khi tạo tài nguyên.
4. Dùng IAM role/user có quyền triển khai trong giai đoạn thiết lập.
5. Tạo GitHub OIDC role ở bước CI/CD thay vì Access Key dài hạn.
6. ACM cho CloudFront được tạo tại `us-east-1`; ACM cho ALB được tạo tại `ap-southeast-1`.

Nên gắn tag `Project=LearnSphere`, `Environment=Production` và `ManagedBy=GitHubActions` cho các tài nguyên hỗ trợ tag để dễ tìm kiếm và theo dõi chi phí.

#### Kiểm soát chi phí

Tạo AWS Budget **trước khi triển khai**:

1. Mở **Billing and Cost Management → Budgets**.
2. Chọn **Create budget → Cost budget**.
3. Chọn chu kỳ Monthly và nhập ngân sách phù hợp.
4. Thêm email cảnh báo tại 50%, 80% và 100%.
5. Xác nhận email nhận cảnh báo.

Hai NAT Gateway, hai EC2 `t3.small`, ALB và CloudWatch Logs là các thành phần cần theo dõi sát vì tạo chi phí theo thời gian hoạt động.

![AWS Budget theo dõi chi phí LearnSphere](/images/learnsphere-aws-budget.png)

*Hình 5.3. AWS Budget giám sát ngân sách 100 USD mỗi tháng và các ngưỡng cảnh báo chi phí của LearnSphere.*

#### Chuẩn bị mã nguồn

![GitHub Actions triển khai LearnSphere thành công](/images/learnsphere-github-actions-overview.png)

*Hình 5.4. GitHub Actions triển khai thành công Backend từ Docker qua ECR tới ASG và Frontend từ bản build qua S3 tới CloudFront trên nhánh `main`.*

```powershell
git clone https://github.com/HoiaeKHMT/LearnSphere.git
cd LearnSphere
git status
git branch --show-current
```

Working tree phải sạch trước khi build; file `.env` không được Git theo dõi.

#### Cấu trúc source code và biến môi trường

LearnSphere là ứng dụng Frontend–Backend dạng container, không sử dụng Lambda, Amazon Cognito hoặc AWS Amplify như một số workshop mẫu. Cấu trúc cần kiểm tra trước khi triển khai:

```text
LearnSphere/
├── LearnSphere_FE/
│   ├── public/
│   ├── src/
│   ├── package.json
│   └── vite.config.ts
├── LearnSphere_BE/
│   ├── src/
│   ├── test/
│   ├── .env.example
│   ├── Dockerfile
│   └── package.json
└── .github/
    └── workflows/
        └── deploy.yml
```

| Thành phần | Cách LearnSphere triển khai |
| --- | --- |
| Frontend | React, TypeScript và Vite; build tĩnh vào `dist`, sau đó đồng bộ lên S3 và phân phối bằng CloudFront |
| Backend | Express.js chạy trong Docker container trên EC2 thuộc Auto Scaling Group |
| Xác thực | JWT lưu trong secure cookie và phân quyền theo role; không dùng Cognito |
| API | Frontend gọi Backend qua đường dẫn CloudFront `/api/*` |
| CI/CD | GitHub Actions dùng OIDC để nhận IAM role, build image lên ECR và triển khai qua SSM/ASG |

Frontend chỉ cần một biến môi trường:

```dotenv
# Chạy local
VITE_API_BASE_URL=http://localhost:5000/api

# Production
VITE_API_BASE_URL=/api
```

`VITE_API_BASE_URL=/api` giúp trình duyệt gọi API cùng origin `www.learnspherev2.id.vn`; CloudFront chuyển tiếp các request `/api/*` đến ALB. Chỉ biến có tiền tố `VITE_` được Vite đưa vào bundle, vì vậy tuyệt đối không đặt MongoDB URI, JWT secret, Groq API key hoặc AWS credential trong cấu hình Frontend.

Tạo file `LearnSphere_BE/.env` từ template `.env.example` cho môi trường local. Các giá trị tối thiểu cần hoàn thiện gồm:

```dotenv
PORT=5000
NODE_ENV=development
TRUST_PROXY=false
MONGODB_URI=<mongodb-atlas-uri>
JWT_SECRET=<strong-random-secret>
FRONTEND_URL=http://localhost:5173
AWS_REGION=ap-southeast-1
AWS_S3_BUCKET=learnsphere-media-2
AI_PROVIDER=groq
GROQ_API_KEY=<groq-api-key>
GROQ_MODEL=<groq-model>
```

Trên production, `NODE_ENV` được đổi thành `production`, `TRUST_PROXY=true`, `FRONTEND_URL=https://www.learnspherev2.id.vn`, và toàn bộ cấu hình Backend được lưu trong Parameter Store SecureString.

#### Chạy ứng dụng local

Mở hai cửa sổ terminal. Terminal thứ nhất chạy Backend:

```powershell
cd LearnSphere_BE
npm ci
npm run dev
```

Terminal thứ hai chạy Frontend:

```powershell
cd LearnSphere_FE
npm ci
npm run dev
```

Truy cập `http://localhost:5173` để kiểm tra giao diện và gọi `http://127.0.0.1:5000/health/ready` để kiểm tra Backend. Nên dùng `npm ci` thay cho `npm install` khi đã có `package-lock.json`, vì lệnh này cài đúng phiên bản dependency đã được khóa và gần với quy trình CI/CD hơn.

#### Kiểm tra mã nguồn local

```powershell
cd LearnSphere_BE
npm ci
npm test

cd ..\LearnSphere_FE
npm ci
npm run build
```

Kiểm tra Docker Backend:

```powershell
cd ..\LearnSphere_BE
docker build -t learnsphere-be:local .
docker run --rm -p 5000:5000 --env-file .env learnsphere-be:local
curl.exe -i http://127.0.0.1:5000/health/ready
```

Kết quả mong đợi:

```json
{"status":"ready","database":"connected"}
```

Ngoài unit test/build, cần xác minh:

* Backend khởi động với `NODE_ENV=production`.
* `/health/live` xác nhận process đang chạy.
* `/health/ready` xác nhận ứng dụng và MongoDB sẵn sàng.
* Frontend sử dụng `VITE_API_BASE_URL=/api`.
* Docker image build được từ Dockerfile trong repository.

![Kết quả kiểm thử Backend LearnSphere](/images/learnsphere-backend-tests.png)

*Hình 5.5a. Backend hoàn thành 15 test case, không có test thất bại.*

![Kết quả build Frontend LearnSphere](/images/learnsphere-frontend-build.png)

*Hình 5.5b. Frontend React/Vite được build thành công và tạo các asset production trong thư mục `dist`.*

Hai ảnh trên kết hợp với response `/health/ready` cho thấy mã nguồn đã vượt qua bước kiểm tra Backend, build Frontend và kết nối database trước khi triển khai lên AWS.

#### Chuẩn bị dịch vụ bên ngoài AWS

| Dịch vụ | Thao tác chuẩn bị |
| --- | --- |
| MongoDB Atlas | Tạo cluster/database user, lưu URI và giới hạn Network Access |
| Groq | Tạo API key, chọn model production và đặt giới hạn sử dụng |
| TenTen DNS | Xác nhận quyền quản lý `learnspherev2.id.vn` |
| Email | Chuẩn bị tài khoản gửi mail nếu kiểm thử chức năng email |

Sau khi NAT Gateway được tạo, MongoDB Atlas chỉ allowlist hai NAT Elastic IP. Không giữ `0.0.0.0/0` trong production.

#### Chuẩn bị cấu hình Backend

Không ghi giá trị secret vào workshop. Chỉ cần kiểm tra đủ các nhóm biến:

| Nhóm | Biến tiêu biểu |
| --- | --- |
| Runtime | `PORT`, `NODE_ENV`, `TRUST_PROXY` |
| Authentication | `JWT_SECRET`, cookie lifetime/domain |
| Database | `MONGODB_URI`, transaction requirement |
| Frontend/CORS | `FRONTEND_URL` |
| Storage | `AWS_REGION`, `AWS_S3_BUCKET`, presigned/multipart settings |
| AI | `AI_PROVIDER=groq`, `GROQ_API_KEY`, `GROQ_MODEL`, rate limit |
| Cleanup | Upload session, S3 cleanup và course retention settings |

File môi trường production sau đó được lưu tại `/learnsphere/prod/backend-env` dưới dạng Parameter Store SecureString.

#### Thông tin cần ghi lại

```text
AWS Account ID: 440893644584
Region: ap-southeast-1
Frontend domain: www.learnspherev2.id.vn
ALB origin domain: origin.learnspherev2.id.vn
ECR repository: learnsphere-be-2
S3 Frontend: learnsphere-fe-2
S3 Media: learnsphere-media-2
```

#### Quy ước đặt tên

| Loại | Tên production |
| --- | --- |
| VPC | `LearnSphere-Prod-vpc` |
| ALB | `LearnSphere-Prod-ALB` |
| Target Group | `LearnSphere-Backend-TG` |
| Launch Template | `LearnSphere-Backend-LT` |
| Auto Scaling Group | `LearnSphere-Backend-ASG` |
| Log group | `/learnsphere/backend2` |
| Environment parameter | `/learnsphere/prod/backend-env` |
| Image parameter | `/learnsphere/prod/backend-image-tag` |

#### Lưu ý bảo mật

Các thông tin nhạy cảm như file `.env`, MongoDB URI, JWT secret, mật khẩu email, Groq API key, cookie và giá trị SecureString phải được lưu trong nơi quản lý cấu hình phù hợp. Không commit các giá trị này lên GitHub và không để chúng xuất hiện trong ảnh báo cáo hoặc log của pipeline.

#### Checklist trước khi tiếp tục

* [ ] AWS Account và hai Region cần dùng đã xác định.
* [ ] MFA và quyền IAM đã kiểm tra.
* [ ] AWS Budget và email cảnh báo đã tạo.
* [ ] Repository `main` đã clone và working tree sạch.
* [ ] Backend test, Frontend build và Docker health check thành công.
* [ ] MongoDB, Groq, domain và email đã sẵn sàng.
* [ ] Tên tài nguyên và tag đã thống nhất.
* [ ] Không có secret nào được commit hoặc xuất hiện trong ảnh minh chứng.
