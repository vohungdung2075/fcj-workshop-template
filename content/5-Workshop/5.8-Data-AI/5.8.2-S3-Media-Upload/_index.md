---
title: "S3 Media and secure upload flow"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.8.2. </b> "
---

#### 1. Separate control and data planes

The backend authorizes and signs requests, while binaries move directly between the browser and S3:

```text
Browser → POST /api/files/presigned-upload → Backend
Browser ← upload URL + session ID           ← Backend
Browser → PUT binary                         → S3 Media
Browser → confirm session                    → Backend
Backend → HEAD object                        → S3 Media
Backend → persist file key                   → MongoDB
```

This avoids buffering 100–200 MB videos in a container and removes large-file bandwidth from ALB/EC2.

#### 2. Single-part upload

Thumbnails, avatars, and small documents use:

```text
POST /api/files/presigned-upload
POST /api/files/profile-avatar/presigned-upload
POST /api/files/uploads/:session_id/confirm
```

The backend validates role, course ownership, folder, MIME type, size, and filename before signing. On confirmation, it uses S3 `HeadObject` to verify metadata before completing the session.

#### 3. Multipart video upload

The frontend selects multipart for videos at least 25 MB:

```text
POST /api/files/multipart/start
→ upload each part through a presigned URL
→ collect ETags
POST /api/files/multipart/:session_id/complete
```

The client uploads up to three parts concurrently and displays progress. If one part fails, it calls:

```text
DELETE /api/files/uploads/:session_id
```

to abort the session instead of leaving an incomplete multipart upload.

#### 4. Downloads and authorization

Private files are accessed through:

```text
GET /api/files/presigned-download
GET /api/files/course-thumbnail/:course_id
GET /api/files/profile-avatar
```

The backend checks enrollment or role before issuing an expiring URL for private files; course thumbnails use a dedicated public endpoint for catalog display. The media bucket still never becomes a public file server.

#### 5. Old-file and orphan cleanup

`UploadSession` tracking and scheduled cleanup:

* Abort expired multipart sessions.
* Delete uploaded objects that were never attached to data.
* Delete an old file only after the database stores its replacement key.
* Delete the course prefix after permanent deletion.
* Retain files during course soft deletion for restore.

The “persist new key before deleting old file” order prevents data loss if S3 or the database fails midway.

#### Deployment evidence

![Top-level prefixes in the private LearnSphere media bucket](/images/learnsphere-s3-media-prefixes.png)

**Figure 5.49:** The private `learnsphere-media-2` bucket separates course assets under `courses/` from user assets under `users/`. The screenshot does not expose a presigned URL or the contents of any private object.

![Multipart video upload progress in the lesson-management interface](/images/learnsphere-multipart-video-upload.png)

**Figure 5.50:** The lesson-management interface reports the progress of a large video uploaded in parts. This gives the tutor visible feedback while the browser completes the transfer instead of waiting on one opaque request.
