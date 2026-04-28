# Day 47 – Advanced Triggers: PR Events, Cron Schedules & Event-Driven Pipelines

## Task
You've used `push` and basic `pull_request` triggers. But GitHub Actions supports **dozens of event types** — today you go deep into PR lifecycle events, scheduled cron jobs, and chaining workflows together.

---

## Challenge Tasks

### Task 1: Pull Request Event Types
Create `.github/workflows/pr-lifecycle.yml` that triggers on `pull_request` with **specific activity types**:
1. Trigger on: `opened`, `synchronize`, `reopened`, `closed`
2. Add steps that:
   - Print which event type fired: `${{ github.event.action }}`
   - Print the PR title: `${{ github.event.pull_request.title }}`
   - Print the PR author: `${{ github.event.pull_request.user.login }}`
   - Print the source branch and target branch
3. Add a conditional step that only runs when the PR is **merged** (closed + merged = true)

Test it: create a PR, push an update to it, then merge it. Watch the workflow fire each time with a different event type.

---

<img width="1044" height="627" alt="image" src="https://github.com/user-attachments/assets/80c7d1d0-ef08-4637-ad3b-0f3e67ff0d18" />

<img width="962" height="636" alt="image" src="https://github.com/user-attachments/assets/aaffb0fd-100b-4cd8-b15d-2624cc747d20" />

---

### Task 2: PR Validation Workflow
Create `.github/workflows/pr-checks.yml` — a real-world PR gate:
1. Trigger on `pull_request` to `main`
2. Add a job `file-size-check` that:
   - Checks out the code
   - Fails if any file in the PR is larger than 1 MB
3. Add a job `branch-name-check` that:
   - Reads the branch name from `${{ github.head_ref }}`
   - Fails if it doesn't follow the pattern `feature/*`, `fix/*`, or `docs/*`
4. Add a job `pr-body-check` that:
   - Reads the PR body: `${{ github.event.pull_request.body }}`
   - Warns (but doesn't fail) if the PR description is empty

**Verify:** Open a PR from a badly named branch — does the check fail?

---

PR check failed since wrong branch name:

<img width="915" height="770" alt="image" src="https://github.com/user-attachments/assets/ab07525f-442f-4270-862a-d74e2764d886" />


---

### Task 3: Scheduled Workflows (Cron Deep Dive)
Create `.github/workflows/scheduled-tasks.yml`:
1. Add a `schedule` trigger with cron: `'30 2 * * 1'` (every Monday at 2:30 AM UTC)
2. Add **another** cron entry: `'0 */6 * * *'` (every 6 hours)
3. In the job, print which schedule triggered using `${{ github.event.schedule }}`
4. Add a step that acts as a **health check** — curl a URL and check the response code

Write in your notes:
- The cron expression for: every weekday at 9 AM IST
- The cron expression for: first day of every month at midnight
- Why GitHub says scheduled workflows may be delayed or skipped on inactive repos

**Important:** Also add `workflow_dispatch` so you can test it manually without waiting for the schedule.

---

schedule.yaml:

    name: Scheduled Tasks
    
    on:
      schedule:
        - cron: '30 2 * * 1'
        - cron: '0 */6 * * *'
      workflow_dispatch:
    
    jobs:
      schedule-job:
        runs-on: ubuntu-latest
    
        steps:
          
          - run: |
             echo "Triggered by: ${{ github.event.schedule }}"
    
          - name: Health check
            run: curl -I https://google.com


Weekday 9 AM IST → 30 3 * * 1-5
1st of month → 0 0 1 * *

---

### Task 4: Path & Branch Filters
Create `.github/workflows/smart-triggers.yml`:
1. Trigger on push but **only** when files in `src/` or `app/` change:
   ```yaml
   on:
     push:
       paths:
         - 'src/**'
         - 'app/**'
   ```
2. Add `paths-ignore` in a second workflow that skips runs when only docs change:
   ```yaml
   paths-ignore:
     - '*.md'
     - 'docs/**'
   ```
3. Add branch filters to only trigger on `main` and `release/*` branches
4. Test it: push a change to a `.md` file — does the workflow skip?

Write in your notes: When would you use `paths` vs `paths-ignore`?

---

