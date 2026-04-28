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


      yaml file:
      
      name: Path & branch filter
      
      on:
        push:
          branches: [main, feature/*]
          paths:
            - 'src/**'
            - 'app/**'
      
          paths-ignore:
              - '*.md'
              - 'docs/**'
      
      jobs:
          test-run:
              runs-on: ubuntu-latest
              steps:
                  - name: sample text
                    run: |
                      echo "Hello world"


---

### Task 5: `workflow_run` — Chain Workflows Together
Create two workflows:
1. `.github/workflows/tests.yml` — runs tests on every push
2. `.github/workflows/deploy-after-tests.yml` — triggers **only after** `tests.yml` completes successfully:
   ```yaml
   on:
     workflow_run:
       workflows: ["Run Tests"]
       types: [completed]
   ```
3. In the deploy workflow, add a conditional:
   - Only proceed if the triggering workflow **succeeded** (`${{ github.event.workflow_run.conclusion == 'success' }}`)
   - Print a warning and exit if it failed

**Verify:** Push a commit — does the test workflow run first, then trigger the deploy workflow?

---


<img width="785" height="493" alt="image" src="https://github.com/user-attachments/assets/cc666fdc-636a-44ba-95eb-bd5f46b7052a" />

---

### Task 6: `repository_dispatch` — External Event Triggers
1. Create `.github/workflows/external-trigger.yml` with trigger `repository_dispatch`
2. Set it to respond to event type: `deploy-request`
3. Print the client payload: `${{ github.event.client_payload.environment }}`
4. Trigger it using `curl` or `gh`:
   ```bash
   gh api repos/<owner>/<repo>/dispatches \
     -f event_type=deploy-request \
     -f client_payload='{"environment":"production"}'
   ```

Write in your notes: When would an external system (like a Slack bot or monitoring tool) trigger a pipeline?

---
      
      name: External Trigger
      
      on:
        repository_dispatch:
          types: [deploy-request]
      
      jobs:
        external:
          runs-on: ubuntu-latest
      
          steps:
            - run: |
                echo "Env: ${{ github.event.client_payload.environment }}"

execute this to run the workflow: gh api repos/vishall24/github-actions-practice-tws/dispatches ` -f event_type=deploy-request ` -f client_payload="{\"environment\":\"production\"}"


workflow_run vs workflow_call
workflow_call → manual reuse (like function call)
workflow_run → automatic chaining after another workflow





## test.yaml:

      name: Run Tests
      
      on: push
      
      jobs:
        test:
          runs-on: ubuntu-latest
          steps:
            - run: echo "Running tests"


## deploy-after-test:

      name: Deploy
      
      on:
        workflow_run:
          workflows: ["Run Tests"]
          types: [completed]
      
      jobs:
        deploy:
          runs-on: ubuntu-latest
      
          steps:
            - name: Check result
              if: github.event.workflow_run.conclusion == 'success'
              run: echo "Deploying..."
      
            - name: Failed case
              if: github.event.workflow_run.conclusion != 'success'
              run: echo "Tests failed, not deploying"



## external:
      
      name: External Trigger
      
      on:
        repository_dispatch:
          types: [deploy-request]
      
      jobs:
        external:
          runs-on: ubuntu-latest
      
          steps:
            - run: |
                echo "Env: ${{ github.event.client_payload.environment }}"



## path_based_publish:

      name: Path & branch filter
      
      on:
        push:
          branches: [main, feature/*]
          paths:
            - 'src/**'
            - 'app/**'
      
          paths-ignore:
              - '*.md'
              - 'docs/**'
      
      jobs:
          test-run:
              runs-on: ubuntu-latest
              steps:
                  - name: sample text
                    run: |
                      echo "Hello world"



## pr-lifecycle.yaml:

      name: PR Lifecycle
      
      on: 
        pull_request:
          types: [opened , synchronize, reopened, closed]
      
      
      jobs:
          pr-info:
              runs-on: ubuntu-latest
      
              steps:
                  - name: Print PR details
                    run: |
                      echo "Event: ${{ github.event.action }}"
                      echo "Title: ${{ github.event.pull_request.title }}"
                      echo "Author: ${{ github.event.pull_request.user.login }}"
                      echo "Source: ${{ github.head_ref }}"
                      echo "Target: ${{ github.base_ref }}"
      
                  - name: Only when PR merged
                    if: github.event.pull_request.merged == true
                    run: echo "PR was merged"


## pr-check.yml:

      name: PR check
      
      on:
        pull_request:
          branches: [main]
      
      jobs:
        pr-job:
          runs-on: ubuntu-latest
      
          steps:
            - name: Print PR info
              run: |
                echo "PR check running for branch == ${{ github.head_ref }}"
      
            - name: Check file size
              run: |
                for file in $(find . -type f); do
                  size=$(du -k "$file" | cut -f1)
                  if [ $size -gt 1024 ]; then
                    echo "File $file too large"
                    exit 1
                  else 
                    echo "less than 1 MB"
                  fi
                done
      
            - name: Check branch name
              run: |
                branch="${{ github.head_ref }}"
                if [[ ! "$branch" =~ ^(feature|fix|docs)/ ]]; then
                  echo "Invalid branch name"
                  exit 1
                fi
      
            - name: Check PR body
              continue-on-error: true
              run: |
                if [ -z "${{ github.event.pull_request.body }}" ]; then
                  echo "PR description is empty"
                  exit 1
                fi
      


## scheduled.yaml:


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



        
