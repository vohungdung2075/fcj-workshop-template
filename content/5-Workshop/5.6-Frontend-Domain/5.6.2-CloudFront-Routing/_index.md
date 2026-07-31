---
title: "CloudFront origins, behaviors, and SPA routing"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.6.2. </b> "
---

#### 1. Two origins, one entry point

Distribution `E3OLXXMB09Q0AJ` uses:

| Origin | Purpose | Connection |
| --- | --- | --- |
| S3 `learnsphere-fe-2` | Static UI | OAC, private bucket |
| `origin.learnspherev2.id.vn` | REST API | HTTPS only |

CloudFront is the browser's single entry point, while the frontend never embeds the ALB origin directly.

#### 2. Configure behaviors

| Path pattern | Origin | Methods/cache |
| --- | --- | --- |
| Default `*` | S3 Frontend | `GET`, `HEAD`; compression; static cache |
| `/api/*` | ALB origin | API HTTP methods; cache disabled |

The `/api/*` behavior forwards the authentication cookie, query strings, and required headers. It must not use a static cache policy because user state and course data vary per request.

#### 3. SPA rewrite

React Router requires `index.html` to handle client routes. A viewer-request CloudFront Function rewrites URIs that end with `/` or whose final segment has no file extension:

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

The function is associated only with the Default behavior that serves the S3 frontend. The higher-priority `/api/*` behavior forwards requests directly to the ALB, so API requests bypass the SPA rewrite and continue to return JSON.

#### 4. Invalidate after release

```bash
aws cloudfront create-invalidation \
  --distribution-id E3OLXXMB09Q0AJ \
  --paths "/*"
```

Invalidation makes edge locations fetch the newest entry point immediately, while hashed assets remain safe for long-term caching.

#### Deployment evidence

![Two production origins configured in CloudFront](/images/learnsphere-cloudfront-origins.png)

*Figure 5.30. CloudFront retains only two production origins: the private S3 bucket for the frontend and the `origin.learnspherev2.id.vn` custom origin backed by the ALB.*

![CloudFront behaviors route frontend and API traffic to the correct origins](/images/learnsphere-cloudfront-behaviors.png)

*Figure 5.31. The `/api/*` behavior routes uncached requests to the ALB, while the Default behavior distributes the frontend from S3.*

![CloudFront Function source used for SPA route rewriting](/images/learnsphere-cloudfront-spa-function.png)

*Figure 5.32. The `LIVE` source of `LearnSphereSpaRewrite2`, which rewrites client-side routes to `index.html`.*
