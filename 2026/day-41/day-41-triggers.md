# Day 41 – Triggers & Matrix Builds

## Task
Your pipeline runs on push. Today you learn **every way to trigger a workflow** and how to run jobs across multiple environments at once.

---

## Challenge Tasks

### Task 1: Trigger on Pull Request
1. Create `.github/workflows/pr-check.yml`
2. Trigger it only when a pull request is **opened or updated** against `main`
3. Add a step that prints: `PR check running for branch: <branch name>`
4. Create a new branch, push a commit, and open a PR
5. Watch the workflow run automatically

**Verify:** Does it show up on the PR page?

---

<img width="1408" height="210" alt="image" src="https://github.com/user-attachments/assets/1c3bff45-35ea-485e-b9b6-e33cf3f599f7" />

When I raised PR from feature-pr branch to main the workflow automatically got triggered.

pr-check.yaml:

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



---

### Task 2: Scheduled Trigger
1. Add a `schedule:` trigger to any workflow using cron syntax
2. Set it to run every day at midnight UTC
3. Write in your notes: What is the cron expression for every Monday at 9 AM?

---

Scheduled for every 5 minutes:
<img width="533" height="280" alt="image" src="https://github.com/user-attachments/assets/789753a1-f1fe-4c81-ba61-60e33aa590ac" />

Cron job for every monday 9 AM:

  - cron: '0 9 * * 1'


---

### Task 3: Manual Trigger
1. Create `.github/workflows/manual.yml` with a `workflow_dispatch:` trigger
2. Add an **input** that asks for an `environment` name (staging/production)
3. Print the input value in a step
4. Go to the **Actions** tab → find the workflow → click **Run workflow**

**Verify:** Can you trigger it manually and see your input printed?

---


<img width="1413" height="661" alt="image" src="https://github.com/user-attachments/assets/8fb9ec36-6136-491b-8e9e-de3d6f5ccf8a" />

manually triggered.

hello.yaml:

    name: First Workflow
    
    on:
      schedule:
        - cron: '*/5 * * * *'
      workflow_dispatch: 
        inputs:
          environment:
            description: "Enter Environment"
            required: true
            default: "staging"
      
    jobs:
      greet_stage_1:
        runs-on: ubuntu-latest
    
        steps:
          - name: Checkout code
            uses: actions/checkout@v4
    
          - name: Say Hello
            run: echo "Hello from Github Actions Runner"
    
          - name: Show Date
            run: date
    
          - name: check who are you in runner
            run: whoami
    
          - name: show current branch
            run: |
              echo 'Branch: "${{ github.ref_name }}"'
    
          - name: List files
            run: ls -la
    
          - name: Show OS
            run: uname -a
    
          - name: Print inputs
            run: |
              echo "Environment is ${{ github.event.inputs.environment }}"

---

### Task 4: Matrix Builds
Create `.github/workflows/matrix.yml` that:
1. Uses a matrix strategy to run the same job across:
   - Python versions: `3.10`, `3.11`, `3.12`
2. Each job installs Python and prints the version
3. Watch all 3 run in parallel

Then extend the matrix to also include 2 operating systems — how many total jobs run now?

---

jobs are running in parallel:

<img width="1304" height="683" alt="image" src="https://github.com/user-attachments/assets/8607ec5e-c7f7-40fd-9b41-3e8ef1b4d19c" />

matrix.yaml:

      name: Matrix Build
      
      on:
        push:
      
      
      jobs:
        build:
          runs-on: ${{ matrix.os }}
      
          strategy:
            matrix:
              os: [ubuntu-latest, windows-latest]
              python-version: [3.11, 3.12, 3.13]
      
          steps:
            - name: Setup python
              uses: actions/setup-python@v4
              with:
                python-version: ${{ matrix.python-version }}
      
            - name: Print python version
              run: python --version


<img width="1152" height="444" alt="image" src="https://github.com/user-attachments/assets/041890fb-c4f2-4d1f-a098-adab43e09cd8" />

---

### Task 5: Exclude & Fail-Fast
1. In your matrix, **exclude** one specific combination (e.g., Python 3.10 on Windows)
2. Set `fail-fast: false` — trigger a failure in one job and observe what happens to the rest
3. Write in your notes: What does `fail-fast: true` (the default) do vs `false`?

---

excluded py:3.11 from windows-latest


<img width="1040" height="590" alt="image" src="https://github.com/user-attachments/assets/e0049d27-5607-4796-a8d3-ac5943a41c94" />

while fail-fast: false 

fail-fast:true   :

<img width="1274" height="754" alt="image" src="https://github.com/user-attachments/assets/6ff08f2e-4958-4720-a10e-a072d2f0bee6" />

fail-fast: true (default)
Ek job fail → sab stop

fail-fast: false
Ek fail → baaki continue


