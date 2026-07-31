---
title: "Deployment preparation"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Prerequisites

| Item | Requirement |
| --- | --- |
| AWS Account | Permission to create IAM, VPC, EC2, ECR, S3, ALB, ASG, ACM, CloudFront, SSM, CloudWatch, and SNS resources |
| Region | `ap-southeast-1`; the CloudFront certificate must be created in `us-east-1` |
| Local tools | Git, Node.js 24, npm, Docker Engine, and AWS CLI v2 |
| Source | Clone LearnSphere and check out the `main` branch |
| Database | A MongoDB Atlas cluster and database user |
| AI | A Groq API key and the production model |
| Domain | `learnspherev2.id.vn` and access to DNS management |

#### Preparation workflow

Preparation begins by validating the AWS Account, IAM permissions, and deployment Regions. The next step is to create an AWS Budget for cost control, clone the `main` branch, and validate the Backend, Frontend, and Docker image locally. Once the source is stable, prepare MongoDB Atlas, the Groq API, the custom domain, and production environment variables. Finally, standardize resource names and tags and complete the security checklist before provisioning AWS infrastructure.

These stages should be completed in order to avoid incurring infrastructure costs while the source, certificates, database, or secrets are still not ready.

#### Account and access preparation

1. Sign in to AWS Account `440893644584` and enable MFA for the root user.
2. Do not use the root user for daily administration.
3. Select Singapore (`ap-southeast-1`) before creating resources.
4. Use an IAM role/user with deployment permissions during initial setup.
5. Create a GitHub OIDC role for CI/CD instead of a long-lived access key.
6. CloudFront ACM resources belong in `us-east-1`; ALB ACM resources belong in `ap-southeast-1`.

Apply `Project=LearnSphere`, `Environment=Production`, and `ManagedBy=GitHubActions` tags where supported to improve resource discovery and cost tracking.

#### Cost control

Create an AWS Budget **before deployment**:

1. Open **Billing and Cost Management → Budgets**.
2. Choose **Create budget → Cost budget**.
3. Select a monthly period and enter an appropriate amount.
4. Add email notifications at 50%, 80%, and 100%.
5. Confirm the notification email.

Two NAT Gateways, two `t3.small` EC2 instances, ALB, and CloudWatch Logs require particular attention because they accrue time-based costs.

![AWS Budget for monitoring LearnSphere costs](/images/learnsphere-aws-budget.png)

*Figure 5.3. AWS Budget monitoring LearnSphere's USD 100 monthly budget and cost alert thresholds.*

#### Prepare the source

![Successful LearnSphere deployment with GitHub Actions](/images/learnsphere-github-actions-overview.png)

*Figure 5.4. GitHub Actions successfully deployed the Backend from Docker through ECR to the ASG and the Frontend build through S3 to CloudFront on the `main` branch.*

```powershell
git clone https://github.com/HoiaeKHMT/LearnSphere.git
cd LearnSphere
git status
git branch --show-current
```

The working tree must be clean before building, and `.env` must not be tracked by Git.

#### Source structure and environment variables

LearnSphere is a containerized Frontend–Backend application. It does not use Lambda, Amazon Cognito, or AWS Amplify as found in some reference workshops. Verify the following structure before deployment:

```text
LearnSphere/
├── LearnSphere_FE/
│   ├── public/
│   ├── src/
│   ├── package.json
│   └── vite.config.ts
├── LearnSphere_BE/
│   ├── src/
│   ├── test/
│   ├── .env.example
│   ├── Dockerfile
│   └── package.json
└── .github/
    └── workflows/
        └── deploy.yml
```

| Component | LearnSphere implementation |
| --- | --- |
| Frontend | React, TypeScript, and Vite; static assets are built into `dist`, synchronized to S3, and delivered through CloudFront |
| Backend | Express.js runs as a Docker container on EC2 instances in an Auto Scaling Group |
| Authentication | JWT in a secure cookie with role-based authorization; Cognito is not used |
| API | The Frontend reaches the Backend through the CloudFront `/api/*` path |
| CI/CD | GitHub Actions assumes an IAM role through OIDC, pushes the image to ECR, and deploys through SSM/ASG |

The Frontend requires only one environment variable:

```dotenv
# Local development
VITE_API_BASE_URL=http://localhost:5000/api

# Production
VITE_API_BASE_URL=/api
```

`VITE_API_BASE_URL=/api` allows the browser to call the API through the same `www.learnspherev2.id.vn` origin, while CloudFront forwards `/api/*` requests to the ALB. Vite exposes variables prefixed with `VITE_` in the client bundle, so MongoDB URIs, JWT secrets, Groq keys, and AWS credentials must never be placed in Frontend configuration.

Create `LearnSphere_BE/.env` from `.env.example` for local development. At minimum, complete the following variables:

```dotenv
PORT=5000
NODE_ENV=development
TRUST_PROXY=false
MONGODB_URI=<mongodb-atlas-uri>
JWT_SECRET=<strong-random-secret>
FRONTEND_URL=http://localhost:5173
AWS_REGION=ap-southeast-1
AWS_S3_BUCKET=learnsphere-media-2
AI_PROVIDER=groq
GROQ_API_KEY=<groq-api-key>
GROQ_MODEL=<groq-model>
```

In production, set `NODE_ENV=production`, `TRUST_PROXY=true`, and `FRONTEND_URL=https://www.learnspherev2.id.vn`, then store the complete Backend configuration in Parameter Store SecureString.

#### Run the application locally

Open two terminal windows. Start the Backend in the first terminal:

```powershell
cd LearnSphere_BE
npm ci
npm run dev
```

Start the Frontend in the second terminal:

```powershell
cd LearnSphere_FE
npm ci
npm run dev
```

Open `http://localhost:5173` to verify the UI and call `http://127.0.0.1:5000/health/ready` to verify the Backend. Prefer `npm ci` over `npm install` when `package-lock.json` exists because it installs the exact locked dependency versions and more closely matches the CI/CD environment.

#### Validate the source locally

```powershell
cd LearnSphere_BE
npm ci
npm test

cd ..\LearnSphere_FE
npm ci
npm run build
```

Validate the Backend image:

```powershell
cd ..\LearnSphere_BE
docker build -t learnsphere-be:local .
docker run --rm -p 5000:5000 --env-file .env learnsphere-be:local
curl.exe -i http://127.0.0.1:5000/health/ready
```

Expected result:

```json
{"status":"ready","database":"connected"}
```

Also verify:

* The Backend starts with `NODE_ENV=production`.
* `/health/live` confirms that the process is running.
* `/health/ready` confirms that the application and MongoDB are ready.
* The Frontend uses `VITE_API_BASE_URL=/api`.
* The Docker image builds from the repository Dockerfile.

![LearnSphere Backend test results](/images/learnsphere-backend-tests.png)

*Figure 5.5a. The Backend completed all 15 test cases with no failures.*

![LearnSphere Frontend build result](/images/learnsphere-frontend-build.png)

*Figure 5.5b. The React/Vite Frontend built successfully and generated production assets in `dist`.*

Together with the `/health/ready` response, these results confirm that Backend tests, the Frontend production build, and database connectivity were validated before AWS deployment.

#### Prepare external services

| Service | Preparation |
| --- | --- |
| MongoDB Atlas | Create the cluster/database user, store the URI, and restrict Network Access |
| Groq | Create an API key, select the production model, and set usage limits |
| TenTen DNS | Confirm management access for `learnspherev2.id.vn` |
| Email | Prepare the sender account if email functionality will be tested |

After NAT Gateways are created, MongoDB Atlas should allowlist only both NAT Elastic IPs. Do not retain `0.0.0.0/0` in production.

#### Prepare Backend configuration

Do not publish secret values in the workshop. Verify only that each variable group is complete:

| Group | Representative variables |
| --- | --- |
| Runtime | `PORT`, `NODE_ENV`, `TRUST_PROXY` |
| Authentication | `JWT_SECRET`, cookie lifetime/domain |
| Database | `MONGODB_URI`, transaction requirement |
| Frontend/CORS | `FRONTEND_URL` |
| Storage | `AWS_REGION`, `AWS_S3_BUCKET`, presigned/multipart settings |
| AI | `AI_PROVIDER=groq`, `GROQ_API_KEY`, `GROQ_MODEL`, rate limits |
| Cleanup | Upload sessions, S3 cleanup, and course retention settings |

The production environment file is later stored in `/learnsphere/prod/backend-env` as a Parameter Store SecureString.

#### Record deployment identifiers

```text
AWS Account ID: 440893644584
Region: ap-southeast-1
Frontend domain: www.learnspherev2.id.vn
ALB origin domain: origin.learnspherev2.id.vn
ECR repository: learnsphere-be-2
S3 Frontend: learnsphere-fe-2
S3 Media: learnsphere-media-2
```

#### Naming convention

| Type | Production name |
| --- | --- |
| VPC | `LearnSphere-Prod-vpc` |
| ALB | `LearnSphere-Prod-ALB` |
| Target Group | `LearnSphere-Backend-TG` |
| Launch Template | `LearnSphere-Backend-LT` |
| Auto Scaling Group | `LearnSphere-Backend-ASG` |
| Log group | `/learnsphere/backend2` |
| Environment parameter | `/learnsphere/prod/backend-env` |
| Image parameter | `/learnsphere/prod/backend-image-tag` |

#### Security considerations

Sensitive information such as `.env` files, MongoDB URIs, JWT secrets, email passwords, Groq API keys, cookies, and SecureString values must remain in an appropriate configuration store. Never commit these values to GitHub or expose them in report screenshots or pipeline logs.

#### Readiness checklist

* [ ] The AWS Account and both required Regions are identified.
* [ ] MFA and IAM permissions are verified.
* [ ] AWS Budget and alert emails are configured.
* [ ] Repository `main` is cloned and the working tree is clean.
* [ ] Backend tests, Frontend build, and Docker health checks succeed.
* [ ] MongoDB, Groq, domain, and email are ready.
* [ ] Resource names and tags are standardized.
* [ ] No secret is committed or shown in evidence screenshots.
