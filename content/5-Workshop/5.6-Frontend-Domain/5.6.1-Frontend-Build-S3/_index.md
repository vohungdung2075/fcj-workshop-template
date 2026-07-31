---
title: "Build and publish the frontend to S3"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.6.1. </b> "
---

#### 1. Prepare the build configuration

The frontend calls the API through the same origin:

```dotenv
VITE_API_BASE_URL=/api
```

This avoids embedding an ALB or CloudFront hostname in the bundle. The browser sends requests to `www.learnspherev2.id.vn/api/*`, and CloudFront selects the backend behavior.

#### 2. Install dependencies and verify the build

```bash
cd LearnSphere_FE
npm ci
npm run build
```

Vite generates `dist/` with `index.html` and hashed CSS, JavaScript, and font files. The build must pass TypeScript checks and contain no backend secrets before upload.

#### 3. Synchronize using separate cache policies

The pipeline publishes three groups:

```bash
aws s3 sync dist/assets/ s3://learnsphere-fe-2/assets/ \
  --cache-control "public,max-age=31536000,immutable"

aws s3 sync dist/ s3://learnsphere-fe-2/ \
  --exclude "assets/*" --exclude "index.html" \
  --cache-control "public,max-age=3600"

aws s3 cp dist/index.html s3://learnsphere-fe-2/index.html \
  --cache-control "no-cache,no-store,must-revalidate"
```

`index.html` is uploaded last so that it never references assets that have not been published. The workflow intentionally avoids `--delete`; lifecycle rules remove old objects without breaking older browser sessions.

#### 4. Protect the bucket

Block Public Access remains enabled. The bucket policy grants `s3:GetObject` only to the CloudFront service principal when `AWS:SourceArn` matches distribution `E3OLXXMB09Q0AJ`. End users never access S3 directly.

#### Deployment evidence

![Frontend objects published to the private learnsphere-fe-2 bucket](/images/learnsphere-frontend-s3-objects.png)

*Figure 5.28. The `learnsphere-fe-2` bucket contains `index.html`, the `assets/` prefix, and the favicon after the frontend build was published.*

![Cache metadata for index.html in S3](/images/learnsphere-s3-index-cache-metadata.png)

*Figure 5.29a. `index.html` uses `no-cache, no-store, must-revalidate` so browsers always check for the latest frontend release.*

![Cache metadata for a hashed JavaScript asset in S3](/images/learnsphere-s3-hashed-asset-cache-metadata.png)

*Figure 5.29b. The hashed JavaScript asset uses `public, max-age=31536000, immutable` for safe long-term caching.*
