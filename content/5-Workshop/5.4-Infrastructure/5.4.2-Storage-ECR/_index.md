---
title: "Amazon S3 and Amazon ECR"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

#### Objective

LearnSphere separates three artifact types into independent resources:

| Resource | Data | Access path |
| --- | --- | --- |
| `learnsphere-fe-2` | React/Vite build | CloudFront Origin Access Control |
| `learnsphere-media-2` | Videos, documents, thumbnails, and avatars | Backend-generated presigned URLs |
| `learnsphere-be-2` | Backend Docker images | GitHub Actions and EC2 IAM roles |

Each data class can therefore use a dedicated policy, lifecycle, and deployment process.

#### 1. Create the Frontend bucket

Create `learnsphere-fe-2` in `ap-southeast-1` with:

* Object Ownership set to Bucket owner enforced.
* All Block Public Access settings enabled.
* Static website hosting not required because CloudFront uses the REST origin.
* Optional versioning for object recovery.
* SSE-S3 or SSE-KMS default encryption.

CloudFront reads the bucket through Origin Access Control. The bucket policy grants `s3:GetObject` to the CloudFront service principal only when `AWS:SourceArn` matches the production distribution.

```bash
aws s3 sync LearnSphere_FE/dist/ s3://learnsphere-fe-2 \
  --delete \
  --region ap-southeast-1
```

The pipeline creates a CloudFront invalidation after synchronization.

#### 2. Create the Media bucket

Create `learnsphere-media-2` in the same Region and keep it private. Objects follow business-oriented prefixes such as:

```text
courses/<courseId>/thumbnail/...
courses/<courseId>/lessons/<lessonId>/videos/...
courses/<courseId>/lessons/<lessonId>/documents/...
users/<userId>/avatars/...
```

The browser never receives AWS credentials. The Backend authenticates the user, validates ownership and role, and returns a time-limited presigned URL. Large videos use multipart upload, while expired sessions and orphaned objects are handled by cleanup tasks.

#### 3. Configure Media CORS

Allowed origins must match the protocol and domain exactly:

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

`ETag` is exposed so that the Frontend can complete multipart uploads. CORS does not replace IAM or bucket policy enforcement; the bucket remains private.

#### 4. Configure protection and lifecycle controls

Verify that:

* Default encryption is enabled on both buckets.
* No public ACL exists.
* Incomplete multipart uploads are aborted after an appropriate retention period.
* Temporary objects and old versions use lifecycle rules when applicable.
* End users do not receive `s3:ListBucket` or object-management permissions.
* The Backend removes the old object only after the database references the new object, and cleans up new orphaned files after database failures.

#### 5. Create the ECR repository

Create the private repository `learnsphere-be-2` in `ap-southeast-1`. Docker images are tagged with the complete Git commit SHA:

```text
440893644584.dkr.ecr.ap-southeast-1.amazonaws.com/learnsphere-be-2:<git-sha>
```

This tag creates a one-to-one relationship between source code and the deployment artifact. The pipeline checks whether the image already exists before building and pushing it.

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

An ECR lifecycle policy can retain recent releases and remove untagged images. The image referenced by `/learnsphere/prod/backend-image-tag` must not be removed.

#### 6. Validate storage and registry

* Confirm that **Block Public Access = On** for both buckets.
* Confirm that the Frontend bucket contains `index.html` and `assets/` after deployment.
* Confirm that Media CORS includes the production origin.
* Confirm that ECR image tags are commit SHAs and inspect their scan status.
* Attempt direct unauthenticated access to a Media object; the request should be denied.

#### Storage and container registry results

![LearnSphere production S3 buckets](/images/learnsphere-s3-buckets.png)

*Figure 5.11. The `learnsphere-fe-2` and `learnsphere-media-2` buckets reside in the Singapore Region (`ap-southeast-1`), separating Frontend build artifacts from learning media.*

![Docker images in Amazon ECR](/images/learnsphere-ecr-images.png)

*Figure 5.12. The `learnsphere-be-2` repository stores Backend Docker images tagged with Git commit SHAs, providing an exact relationship between each released artifact and its source revision.*

The result separates static artifacts, media objects, and Backend container images into independent storage boundaries. CI/CD updates only the required resource, while a previous commit-SHA image remains available for Backend rollback.
