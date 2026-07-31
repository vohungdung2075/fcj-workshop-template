---
title: "Frontend, CloudFront, and custom domain"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Objectives

This section deploys the LearnSphere frontend delivery layer after the highly available backend is ready. React/Vite is built into static assets, stored in a private S3 bucket, and distributed through CloudFront. The website domain handles both the UI and `/api/*`, keeping authentication cookies consistent and preventing the frontend from depending directly on the ALB DNS name.

Expected results:

* Production uses `VITE_API_BASE_URL=/api`.
* `learnsphere-fe-2` remains private and is readable only through CloudFront OAC.
* Hashed assets are cached for a long period, while the API and `index.html` are not cached.
* Direct SPA route refreshes are rewritten to `/index.html`.
* `www.learnspherev2.id.vn` serves the website with valid HTTPS.
* `origin.learnspherev2.id.vn` provides a valid HTTPS backend origin.

#### Production components

| Component | Configuration |
| --- | --- |
| Frontend | React, TypeScript, Vite |
| Frontend bucket | `learnsphere-fe-2` |
| CDN | CloudFront distribution `E3OLXXMB09Q0AJ` |
| Website | `https://www.learnspherev2.id.vn` |
| Backend origin | `https://origin.learnspherev2.id.vn` |
| CloudFront TLS | ACM in `us-east-1` |
| ALB TLS | ACM in `ap-southeast-1` |
| DNS | TenTen CNAME records |

#### Deployment steps

1. [Build the frontend and publish it to S3](5.6.1-frontend-build-s3/)
2. [Configure CloudFront origins, behaviors, and SPA routing](5.6.2-cloudfront-routing/)
3. [Configure ACM and DNS](5.6.3-acm-dns/)
4. [Validate the production website](5.6.4-production-validation/)

#### Production request flow

```text
Browser
→ https://www.learnspherev2.id.vn
→ CloudFront
   ├─ /*       → S3 Frontend through OAC
   └─ /api/*   → HTTPS ALB origin
                  → Target Group
                  → healthy Backend EC2
```

The two behaviors use different cache policies. Static assets are optimized at edge locations, whereas API requests are forwarded to the backend with the required cookies and request data. This keeps the UI fast without serving stale dynamic data.

#### Completion criteria

Section 5.6 is complete when the custom domain serves the website over HTTPS, direct SPA route refreshes do not return 404, `/api/*` reaches the ALB, the frontend bucket remains private, and a new release becomes visible after CloudFront invalidation.
