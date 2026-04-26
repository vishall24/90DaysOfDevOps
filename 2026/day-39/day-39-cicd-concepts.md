## The Problem

1. What can go wrong?

- Code conflicts between developers
- Bugs go to production
- Manual mistakes during deployment
- No proper testing
- Version confusion

2. "It works on my machine"

This means code runs on developer's system but fails in production due to different environments.

3. Manual deployment frequency

Very limited (1–2 times per day max), risky and slow.

TTM ( Time to market ) is very high.

## CI vs CD

### Continuous Integration (CI)

Developers frequently push code. Code is automatically built and tested.

Example: Push → tests run → build fails if bug exists

---

### Continuous Delivery (CD)
Code is ready to deploy anytime, but deployment is manual.

Example: After tests pass → ready → developer clicks deploy

---

### Continuous Deployment
Code is automatically deployed after passing tests.

Example: Push → test → build → auto deploy to production

## Pipeline Anatomy

- Trigger → Starts pipeline (push, PR)
- Stage → Group of jobs (build, test, deploy)
- Job → Task inside stage (run tests)
- Step → Single command (npm install)
- Runner → Machine executing jobs
- Artifact → Output (build files, Docker image)

## Pipeline Diagram

Developer pushes code → GitHub

Stage 1: Build
- Install dependencies
- Build app

Stage 2: Test
- Run unit tests

Stage 3: Deploy
- Build Docker image
- Deploy to staging server

## Real World Pipeline

Repository: Node.js (example)

Trigger:
On cron job every midnight 12 AM 

Jobs:
Multiple jobs (checkout code + build + test)

What it does:
- Installs dependencies
- Runs tests
- Checks code quality

CI/CD is an automation of code → build → test → deploy.
