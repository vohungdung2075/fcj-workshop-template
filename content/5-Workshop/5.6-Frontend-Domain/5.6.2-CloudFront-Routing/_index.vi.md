---
title: "CloudFront origins, behaviors và SPA routing"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.6.2. </b> "
---

#### 1. Hai origin, một entry point

Distribution `E3OLXXMB09Q0AJ` sử dụng:

| Origin | Mục đích | Kết nối |
| --- | --- | --- |
| S3 `learnsphere-fe-2` | Giao diện tĩnh | OAC, bucket private |
| `origin.learnspherev2.id.vn` | REST API | HTTPS only |

CloudFront trở thành entry point duy nhất cho trình duyệt, còn ALB origin không được ghi trực tiếp vào Frontend.

#### 2. Cấu hình behavior

| Path pattern | Origin | Phương thức/cache |
| --- | --- | --- |
| Default `*` | S3 Frontend | `GET`, `HEAD`; nén; cache static |
| `/api/*` | ALB origin | các HTTP method API; cache disabled |

Behavior `/api/*` phải forward cookie xác thực, query string và các header cần thiết. API không được dùng static cache policy vì trạng thái người dùng và dữ liệu khóa học thay đổi theo request.

#### 3. SPA rewrite

React Router cần `index.html` xử lý route phía client. CloudFront Function tại viewer request đổi URI kết thúc bằng `/` hoặc có thành phần cuối không chứa phần mở rộng:

```javascript
function handler(event) {
  var request = event.request;
  var uri = request.uri;
  var lastPart = uri.split("/").pop();

  if (uri.endsWith("/") || !lastPart.includes(".")) {
    request.uri = "/index.html";
  }

  return request;
}
```

Function chỉ được liên kết với behavior Default phục vụ S3 Frontend. Behavior `/api/*` có độ ưu tiên cao hơn và chuyển request thẳng đến ALB, vì vậy API không đi qua SPA rewrite và vẫn trả về JSON.

#### 4. Invalidation sau phát hành

```bash
aws cloudfront create-invalidation \
  --distribution-id E3OLXXMB09Q0AJ \
  --paths "/*"
```

Invalidation giúp edge lấy `index.html` và cấu hình mới ngay sau deploy; asset có hash vẫn an toàn khi cache dài hạn.

#### Bằng chứng triển khai

![Hai origin production được cấu hình trong CloudFront](/images/learnsphere-cloudfront-origins.png)

*Hình 5.30. CloudFront chỉ duy trì hai origin production: bucket S3 private cho Frontend và custom origin `origin.learnspherev2.id.vn` trỏ đến ALB.*

![Hai CloudFront behavior định tuyến Frontend và API đến đúng origin](/images/learnsphere-cloudfront-behaviors.png)

*Hình 5.31. Behavior `/api/*` chuyển request đến ALB với cache bị tắt; Default behavior phân phối Frontend từ S3.*

![Mã CloudFront Function dùng để rewrite route SPA](/images/learnsphere-cloudfront-spa-function.png)

*Hình 5.32. Mã `LIVE` của CloudFront Function `LearnSphereSpaRewrite2` dùng để chuyển các route phía client về `index.html`.*
