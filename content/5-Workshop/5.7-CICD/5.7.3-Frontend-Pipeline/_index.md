---
title: "Frontend release pipeline"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.7.3. </b> "
---

#### 1. Depend on the backend release

```yaml
deploy-frontend:
  needs: deploy-backend
```

The frontend runs only after backend tests, rollout, and public health validation succeed. This prevents publishing a UI that calls an unavailable API.

#### 2. Validate the API base URL

The pipeline accepts:

* `/api` for same-origin production; or
* a valid HTTPS URL for another environment.

An HTTP or malformed value fails the job before the build starts.

#### 3. Build and synchronize S3

```bash
cd LearnSphere_FE
npm ci
npm run build
```

Hashed assets receive one-year immutable caching, other files use a one-hour cache, and `index.html` is uploaded last with `no-cache,no-store,must-revalidate`.

No AWS credentials enter the Vite environment. OIDC exists only in the runner for S3 and CloudFront operations.

#### 4. Invalidate CloudFront

After successful upload:

```bash
aws cloudfront create-invalidation \
  --distribution-id "$CLOUDFRONT_FE_DISTRIBUTION_ID" \
  --paths "/*"
```

The job completes after AWS accepts the invalidation. Users then receive the new entry point, while content-hashed assets avoid cache collisions.

#### Deployment evidence

![Completed frontend deployment steps in GitHub Actions](/images/learnsphere-frontend-deploy-job.png)

*Figure 5.43. The frontend job completes dependency installation, production bundle creation, S3 synchronization, and CloudFront invalidation.*

![CloudFront invalidations following frontend pipeline releases](/images/learnsphere-cicd-cloudfront-invalidations.png)

*Figure 5.44. CloudFront records the invalidations as `Completed`, confirming that edge caches were refreshed after frontend releases.*
