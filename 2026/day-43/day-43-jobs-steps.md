# Day 43 – Jobs, Steps, Env Vars & Conditionals

## Task
Today you learn how to **control the flow** of your pipeline — multi-job workflows, passing data between jobs, environment variables, and running steps only when certain conditions are met.

---

## Challenge Tasks

### Task 1: Multi-Job Workflow
Create `.github/workflows/multi-job.yml` with 3 jobs:
- `build` — prints "Building the app"
- `test` — prints "Running tests"
- `deploy` — prints "Deploying"

Make `test` run only **after** `build` succeeds.
Make `deploy` run only **after** `test` succeeds.

**Verify:** Check the workflow graph in the Actions tab — does it show the dependency chain?

---

<img width="1393" height="673" alt="image" src="https://github.com/user-attachments/assets/be6e5adc-4feb-47cc-84e9-81a44d844291" />

needs: controls execution order
Flow: build → test → deploy

---

### Task 2: Environment Variables
In a new workflow, use environment variables at 3 levels:
1. **Workflow level** — `APP_NAME: myapp`
2. **Job level** — `ENVIRONMENT: staging`
3. **Step level** — `VERSION: 1.0.0`

Print all three in a single step and verify each is accessible.

Then use a **GitHub context variable** — print the commit SHA and the actor (who triggered the run).

---

Output:

<img width="1470" height="758" alt="image" src="https://github.com/user-attachments/assets/641351b0-2658-492c-bcc6-a79417767c25" />

Scope hierarchy: step > job > workflow
GitHub gives built-in variables

---

### Task 3: Job Outputs
1. Create a job that **sets an output** — e.g., today's date as a string
2. Create a second job that **reads that output** and prints it
3. Pass the value using `outputs:` and `needs.<job>.outputs.<name>`

Write in your notes: Why would you pass outputs between jobs?

---

<img width="1366" height="578" alt="image" src="https://github.com/user-attachments/assets/aa70e49f-3fd1-444c-9044-29ee79b35023" />

<img width="1174" height="582" alt="image" src="https://github.com/user-attachments/assets/14a1ae75-8262-4d5d-95b7-a2c52f313b85" />

One job → produces data
Another job → consumes it

Used in real pipelines for:

passing build versions
passing image tags

---

### Task 4: Conditionals
In a workflow, add:
1. A step that only runs when the branch is `main`
2. A step that only runs when the previous step **failed**
3. A job that only runs on **push** events, not on pull requests
4. A step with `continue-on-error: true` — what does this do?

---

<img width="1127" height="855" alt="image" src="https://github.com/user-attachments/assets/468fb2f5-0ee0-4201-b536-475dfe151be9" />

<img width="1054" height="513" alt="image" src="https://github.com/user-attachments/assets/2f94cd5e-e8ba-409e-a275-7988dc659c3c" />

if: controls execution
failure() checks previous step
continue-on-error: true → pipeline doesn’t stop

---

### Task 5: Putting It Together
Create `.github/workflows/smart-pipeline.yml` that:
1. Triggers on push to any branch
2. Has a `lint` job and a `test` job running in parallel
3. Has a `summary` job that runs after both, prints whether it's a `main` branch push or a feature branch push, and prints the commit message

---

<img width="1220" height="633" alt="image" src="https://github.com/user-attachments/assets/86684ef6-57d6-448e-8b17-70e14963d308" />

from feature branch:


<img width="1001" height="603" alt="image" src="https://github.com/user-attachments/assets/a2f8faaf-5b44-445c-ac1b-560fc1310e58" />


## smart-pipeline.yaml:

    name: Smart Pipeline
    
    on:
      push:
    
    jobs:
      lint:
        runs-on: ubuntu-latest
        steps:
          - run: echo "Linting code"
            
      test:
        runs-on: ubuntu-latest
        steps:
          - run: echo "Running tests"
    
      summary:
        runs-on: ubuntu-latest
        needs: [lint, test]
    
        steps:
            - name: branch type
              run: |
                if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
                   echo "Main branch push"
    
                else
                   echo "Feature branch pushed"
                fi
    
            - name: Commit message
              run: |
                echo "${{ github.event.commits[0].message }}"


## output.yaml:

    name: Job outputs
    
    on: push
    
    jobs:
        job_1:
            runs-on: ubuntu-latest
            outputs:
                today: ${{ steps.set-data.outputs.data}}
    
            steps:
                - name: Set data
                  id: set-data
                  run: |
                    echo "data=$(date)" >> $GITHUB_OUTPUT
    
    
        job_2:
            runs-on: ubuntu-latest
            needs: job_1
    
            steps:
                - name: Read Output
                  run: echo "Date is ${{ needs.job_1.outputs.today}}"


  ## multi-job.yaml:

      name: Multi Job pipelines
    
      on:
        push:
      
      
      jobs:
          build: 
              runs-on: ubuntu-latest
              steps:
                  - run: |
                      echo "Building app"
      
          test:
              runs-on: ubuntu-latest
              steps:
                  - run: echo "Running tests"    
      
      
          deploy:
              runs-on: ubuntu-latest
              needs: test
              steps:
              - run: echo "Deploying"



## env.yaml:

    name: Env vars
    
    on: 
        push:
    
    
    env:
        APP_NAME: myapp
    
    
    jobs:
        env-jobs:
            runs-on: ubuntu-latest
    
            env: 
                ENVIRONMENT: staging
            
            steps:
                - name: Print all Vars
                  env:
                    VERSION: 1.0.0
                  run: |
                    echo "App : $APP_NAME"
                    echo "Env : $ENVIRONMENT"
                    echo "Version: $VERSION"
    
                - name: GITHUB context
                  run: |
                    echo "Commit : ${{ github.sha }}"
                    echo "Triggered by ${{ github.actor }}"



## conditions.yaml:
    
    name: Conditions
    
    on:
        push:
        pull_request:
    
    jobs:
        conditional-jobs:
            runs-on: ubuntu-latest
    
            steps:
                - name: Run only on main
                  if: github.ref == 'refs/heads/main'
                  run: echo "Running step on main branch"
    
                - name: Fail step
                  run: exit 1
    
                - name: Run if previous failed
                  if: failure()
                  run: echo "Previous step failed....."
    
                - name: Continue even if error
                  run: exit 1
                  continue-on-error: true
    
        only-push-job:
            if: github.event_name == 'push'
            runs-on: ubuntu-latest
            steps:
                - run: |
                    echo "This runs only on "push" "
