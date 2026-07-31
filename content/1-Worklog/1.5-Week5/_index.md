---
title: "Week 5 Worklog"
date: 2026-06-29
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 objectives

* Analyze the LearnSphere architecture, handover status, and business requirements.
* Complete the core Backend features for accounts, courses, lessons, and quizzes.
* Implement course enrollment, learning progress, and role-based notification workflows.
* Ensure the safe management of MongoDB data and learning materials stored on Amazon S3.

### Tasks completed during the week

| Day | Tasks | Start date | Completion date | Reference material |
| --- | --- | --- | --- | --- |
| **2** | - Read the handover documentation and API Design and inspected the actual LearnSphere structure.<br>- Analyzed the React/Vite Frontend, Express.js Backend, and MongoDB database architecture.<br>- Reviewed the existing models, routes, controllers, services, and environment variables.<br>- Identified completed and missing features and prepared the development plan for the final four weeks. | 06/29/2026 | 06/29/2026 | https://react.dev/learn/creating-a-react-app<br>https://expressjs.com/en/guide/routing.html<br>https://www.mongodb.com/docs/manual/contents/ |
| **3** | - Completed the authentication and account management workflows:<br>&emsp;+ Registration and login<br>&emsp;+ JWT and authentication cookies<br>&emsp;+ Forgot and reset password<br>&emsp;+ Student, tutor, and admin authorization<br>&emsp;+ Pending, active, and blocked account statuses<br>- Added profile and avatar update APIs.<br>- Reviewed authentication and authorization middleware and 401/403 error responses. | 06/30/2026 | 06/30/2026 | https://expressjs.com/en/advanced/best-practice-security.html<br>https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies |
| **4** | - Completed the course and lesson management APIs.<br>- Implemented course creation, retrieval, update, soft deletion, restoration, and permanent deletion.<br>- Completed lesson CRUD, ordering, and completion progress tracking.<br>- Stored videos, documents, and thumbnails as S3 object keys instead of temporary URLs.<br>- Added safe S3 cleanup when files or records are replaced or deleted. | 07/01/2026 | 07/01/2026 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html<br>https://www.mongodb.com/docs/manual/core/transactions/ |
| **5** | - Completed the quiz and question workflows:<br>&emsp;+ Create, update, and delete quizzes<br>&emsp;+ Manage questions and answers<br>&emsp;+ Start an attempt<br>&emsp;+ Enforce deadlines and expiration<br>&emsp;+ Submit, score, and store attempt history<br>- Stored question snapshots when attempts started so later quiz edits would not affect results.<br>- Prevented quiz editing while students had valid in-progress attempts. | 07/02/2026 | 07/02/2026 | https://www.mongodb.com/docs/manual/core/transactions/<br>https://mongoosejs.com/docs/validation.html |
| **6** | - Completed enrollment workflows for open and approval-required courses.<br>- Added student enrollment and cancellation and tutor approval, rejection, and removal actions.<br>- Preserved pending requests when a tutor changed a course to open enrollment.<br>- Completed My Courses, course progress, and quiz performance APIs.<br>- Reviewed course notifications and discussion features.<br>- Updated the API documentation to match the completed implementation. | 07/03/2026 | 07/03/2026 | https://www.mongodb.com/docs/manual/data-modeling/<br>https://expressjs.com/en/guide/routing.html |

### Week 5 achievements

* Gained a clear understanding of the LearnSphere architecture and handover status.

* Completed authentication, profile updates, and authorization for:
  * Students
  * Tutors
  * Administrators
* Completed the course lifecycle, including creation, updates, soft deletion, restoration, and permanent deletion.
* Completed lesson CRUD, S3 learning-material management, and learning progress tracking.
* Completed timed quiz attempts with question snapshots, scoring, and attempt history.
* Implemented enrollment, cancellation, approval, and course-member management workflows.
* Ensured that changing the enrollment type did not remove pending student requests.
* Completed course progress, quiz results, notifications, and discussion APIs.
* Synchronized the API documentation with the Backend implementation.
