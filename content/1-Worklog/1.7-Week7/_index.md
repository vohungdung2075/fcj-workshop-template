---
title: "Week 7 Worklog"
date: 2026-07-13
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 objectives

* Complete the LearnSphere interface for student, tutor, and administrator roles.
* Connect the Frontend with course, lesson, quiz, AI, and enrollment APIs.
* Improve the content management, learning, and quiz-taking experience.
* Test and fix business logic and prepare the application for production.
* Prepare and verify the application before deploying it to AWS.

### Tasks completed during the week

| Day | Tasks | Start date | Completion date | Reference material |
| --- | --- | --- | --- | --- |
| **2** | - Completed the application shell, Header, Sidebar, and role-based navigation.<br>- Added protected routes, a 404 page, redirects, and SPA navigation.<br>- Completed the profile page, avatar updates, and loading, success, and error toasts.<br>- Improved responsive layouts and enlarged the Sphere AI window for better readability. | 07/13/2026 | 07/13/2026 | https://react.dev/learn<br>https://reactrouter.com/start/declarative/routing |
| **3** | - Completed the course catalog, course detail, and course management pages.<br>- Added search, filters, popular courses, and an automatic carousel.<br>- Improved thumbnail display, active-student counts, and enrollment statuses.<br>- Separated the course overview from the first lesson.<br>- Completed course create, update, delete, and restore forms and the lesson management interface. | 07/14/2026 | 07/14/2026 | https://react.dev/learn<br>https://vite.dev/guide/ |
| **4** | - Completed the lesson detail page and module list.<br>- Allowed tutors to create a lesson and navigate directly to the newly created lesson.<br>- Integrated video, document, and thumbnail uploads with upload progress.<br>- Highlighted lesson documents and limited AI document status visibility to tutors and administrators.<br>- Integrated document summaries, mathematical expression rendering, and large-video loading states. | 07/15/2026 | 07/15/2026 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html<br>https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video |
| **5** | - Reorganized quiz management into course selection, quiz selection, question creation, and review steps.<br>- Integrated AI question generation by question count and difficulty.<br>- Moved the AI draft action to the end of the question list so tutors could review before saving.<br>- Completed the quiz player, timer, latest-answer auto-submit, and duplicate-submit protection.<br>- Added KaTeX math rendering and retry behavior for failed submissions. | 07/16/2026 | 07/16/2026 | https://katex.org/docs/supported<br>https://react.dev/learn/updating-arrays-in-state |
| **6** | - Completed student and tutor enrollment interfaces.<br>- Added cancellation, approval, rejection, and course-member removal actions.<br>- Created a Student Learning Report modal showing progress and detailed quiz attempts when a tutor selected a student.<br>- Adjusted dashboards by role and completed administrator user, course, quiz, and system-monitoring pages.<br>- Ran Frontend and Backend checks, fixed business-logic issues, and prepared the production configuration. | 07/17/2026 | 07/17/2026 | https://react.dev/learn<br>https://www.mongodb.com/docs/manual/aggregation/ |

### Week 7 achievements

* Completed SPA navigation, protected routes, and role-based interfaces.

* Completed the profile page and avatar upload workflow.
* Improved course discovery with search, filters, a popular-course carousel, and accurate student counts.
* Completed course, lesson, and learning-material management workflows.
* Fully integrated AI chat, document summaries, and question generation into the Frontend.
* Completed quiz management and quiz-taking interfaces with safe timer-based auto-submission.
* Completed enrollment workflows for students and tutors.
* Displayed individual student progress and detailed quiz performance reports.
* Completed administrator management pages.
* Verified builds, fixed business-logic issues, and prepared the application for production.
