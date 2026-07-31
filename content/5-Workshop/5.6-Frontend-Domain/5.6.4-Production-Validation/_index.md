---
title: "Validate the production frontend"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.6.4. </b> "
---

#### 1. Validate DNS and HTTPS

```powershell
nslookup www.learnspherev2.id.vn
curl.exe -I https://www.learnspherev2.id.vn
curl.exe -i https://www.learnspherev2.id.vn/api/courses
curl.exe -i https://origin.learnspherev2.id.vn/health/ready
```

Expected results:

* The domain resolves to CloudFront.
* The website returns HTTP 200 over HTTPS.
* `/api/courses` is forwarded to Express and returns JSON.
* Origin `/health/ready` returns `{"status":"ready","database":"connected"}`.
* The browser reports no certificate or mixed-content error.

#### 2. Validate SPA routing

Open and refresh routes such as:

```text
/profile
/courses
/system-monitoring
/lesson-management
```

Each route must reload the application instead of returning an S3 403/404. A non-existent API URL must return Express JSON 404 rather than `index.html`.

#### 3. Validate cache and release visibility

After deployment:

1. Confirm that the CloudFront invalidation has completed.
2. Use an incognito window to avoid an old browser cache.
3. Inspect the new asset filenames in Developer Tools.
4. Sign in and call an authenticated API.
5. Confirm that presigned uploads still satisfy S3 CORS.

#### Deployment evidence

![LearnSphere running on the production domain](/images/learnsphere-production-homepage.png)

*Figure 5.36. LearnSphere is delivered over HTTPS in production. The application UI confirms that CloudFront serves the frontend from S3 and that users can access it through a browser.*


![Completed CloudFront invalidations](/images/learnsphere-cloudfront-invalidations.png)

*Figure 5.37. The CloudFront invalidation history shows that all edge-cache refresh requests completed successfully.*

![Production API request routed through the same domain with a 200 OK response](/images/learnsphere-production-api-network.png)

*Figure 5.38. The `GET /api/users/me/courses` request is sent through `https://www.learnspherev2.id.vn` and returns `200 OK`, confirming successful CloudFront-to-backend API routing.*
