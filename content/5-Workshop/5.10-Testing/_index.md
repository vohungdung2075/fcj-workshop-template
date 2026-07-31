---
title: "Testing and results"
date: 2026-07-30
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

#### Test scope and entry conditions

Production acceptance begins only after the latest GitHub Actions workflow succeeds, CloudFront deployment finishes, the ALB has two healthy targets, and the tester has one account for each role. Tests use sample learning data rather than personal or confidential documents.

The test strategy combines:

| Test level | Main objective |
| --- | --- |
| Automated backend tests | Validate security headers, CORS, readiness, environment validation, and AI quiz parsing |
| Build validation | Confirm the React/Vite production bundle compiles |
| Infrastructure smoke test | Confirm DNS, TLS, CloudFront, ALB, Target Group, ASG, and MongoDB readiness |
| Role-based acceptance | Validate complete Student, Tutor, and Administrator workflows |
| Failure-path test | Verify authorization, invalid upload, AI error, and HA recovery behavior |

#### Production acceptance matrix

| Area | Test | Expected result |
| --- | --- | --- |
| DNS/TLS | Open `https://www.learnspherev2.id.vn` | Valid certificate |
| Frontend | Refresh a nested SPA route | No 404 response |
| API readiness | Call `https://origin.learnspherev2.id.vn/health/ready` | HTTP 200 with `database: connected` |
| ALB | Inspect Target Health | Two healthy targets |
| Multi-AZ | Inspect ASG instances | At least one instance per AZ |
| MongoDB | Inspect health response | `database: connected` |
| Media | Upload/view thumbnail, document, video | Presigned URLs work |
| AI | Summarize a document and generate a quiz | Valid Groq output |
| CI/CD | Push a commit to `main` | Both jobs succeed |

Each test record should contain the build commit, execution time, actor role, expected result, actual result, and evidence. A test is `Pass` only when the expected business result and security boundary are both satisfied.

#### Role-based end-to-end tests

**Student**

1. Register and sign in.
2. Request enrollment or join an open course.
3. View lessons, videos, and documents.
4. Use the lesson-context AI Assistant.
5. Complete quizzes and review results.
6. Cancel course enrollment.

**Tutor**

1. Create courses and lessons.
2. Upload thumbnails, videos, and documents.
3. Approve/remove students and inspect progress/submission details.
4. Create quizzes manually or with AI difficulty.
5. Soft-delete and restore a course.

**Administrator**

1. Manage accounts.
2. Review courses/quizzes by tutor without editing their content.
3. Open System Monitoring.
4. Review notifications and system status.

The role tests also verify negative authorization: students cannot enter tutor/admin management routes, tutors cannot administer accounts, and administrators can review tutor-owned course/quiz data without silently becoming its author.

#### Failure-path tests

| Scenario | Expected response |
| --- | --- |
| Request without authentication | `401` and no protected data |
| Authenticated user with the wrong role | `403` |
| Unsupported or oversized upload | Specific validation error; no orphan object |
| Expired presigned URL | Upload/download rejected; request a new URL |
| Groq quota or timeout | Normalized AI error and usable non-AI page |
| Invalid structured quiz response | Draft rejected; no malformed questions persisted |
| MongoDB disconnected | Readiness fails and target leaves ALB rotation |

#### High Availability validation

1. Confirm two healthy Target Group targets.
2. Manually terminate one ASG instance.
3. Call the health endpoint and use the site during replacement.
4. Confirm ALB continues serving through the remaining target.
5. Wait for ASG to create a replacement from the default Launch Template.
6. Confirm two healthy targets across two AZs again.

The termination test must run during a monitored window. `min` and `desired` remain at `2`; otherwise the experiment would change the availability requirement being validated. Success requires continuous health responses through the remaining target and a return to two healthy targets without manual installation.

#### Recorded results

The production health endpoint returned:

```http
HTTP/1.1 200 OK
```

```json
{"status":"ready","database":"connected"}
```

The backend automated suite completed 15 tests with no failures, and the frontend production build completed successfully. GitHub Actions completed both Backend and Frontend deployment jobs. The ASG maintained two instances across two Availability Zones, while the Target Group reported two healthy targets.

#### Exit criteria

Production is accepted when there are no unresolved critical/high defects; DNS, TLS, API and nested SPA routes pass; both targets are healthy; the three role journeys complete; media and AI workflows succeed; and the deployment can be traced to a successful commit. Any failed security, data-loss, or HA test blocks release.

#### Acceptance evidence

![Production API returns HTTP 200](/images/learnsphere-production-api-patch-200.png)

*Figure 5.58. DevTools records `PATCH /api/users/me` on the production domain `https://www.learnspherev2.id.vn` returning `HTTP 200 OK`. This result demonstrates that CloudFront correctly forwards the `/api/*` flow to the ALB and that the Backend successfully processes an authenticated request.*

![Auto Scaling Group EC2 replacement history](/images/learnsphere-asg-instance-refresh-activity.png)

*Figure 5.59. The `LearnSphere-Backend-ASG` activity history records successful launches of replacement instances and termination of the previous instances during Instance Refresh. Together with the two healthy Target Group targets shown in Figure 5.8, this confirms that the Backend supports rolling deployment and automated recovery without manually configuring each EC2 instance.*
