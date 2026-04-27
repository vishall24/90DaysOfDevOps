<img width="1978" height="1210" alt="image" src="https://github.com/user-attachments/assets/8a85670b-7d8f-44ee-94dc-45c7c7ab5029" /># Day 44 – Secrets, Artifacts & Running Real Tests in CI

## Task
Today your pipeline starts doing **real work** — storing sensitive values securely, saving build outputs, and running actual tests from your previous days.

---

## Challenge Tasks

### Task 1: GitHub Secrets
1. Go to your repo → Settings → Secrets and Variables → Actions
2. Create a secret called `MY_SECRET_MESSAGE`
3. Create a workflow that reads it and prints: `The secret is set: true` (never print the actual value)
4. Try to print `${{ secrets.MY_SECRET_MESSAGE }}` directly — what does GitHub show?

Write in your notes: Why should you never print secrets in CI logs?

---


<img width="1978" height="1210" alt="image" src="https://github.com/user-attachments/assets/259d1f9e-01e5-4857-b5f7-3a1ddf9b544c" />

Token leak = account compromise

---

### Task 2: Use Secrets as Environment Variables
1. Pass a secret to a step as an environment variable
2. Use it in a shell command without ever hardcoding it
3. Add `DOCKER_USERNAME` and `DOCKER_TOKEN` as secrets (you'll need these on Day 45)

---

secrets.yaml:

      - name: Use secret as env
        env:
            SECRET_MSG: ${{ secrets.MY_SECRET_MESSAGE }}
        run: echo "Using secret safely"

   
---

### Task 3: Upload Artifacts
1. Create a step that generates a file — e.g., a test report or a log file
2. Use `actions/upload-artifact` to save it
3. After the workflow runs, download the artifact from the Actions tab

**Verify:** Can you see and download it from GitHub?

---

<img width="2670" height="1350" alt="image" src="https://github.com/user-attachments/assets/0627da4d-22df-412c-a536-b687c65b75be" />


<img width="1366" height="848" alt="image" src="https://github.com/user-attachments/assets/8963639a-ed3a-4f41-a259-084f949a3e9e" />

---

### Task 4: Download Artifacts Between Jobs
1. Job 1: generate a file and upload it as an artifact
2. Job 2: download the artifact from Job 1 and use it (print its contents)

Write in your notes: When would you use artifacts in a real pipeline?

---

<img width="2610" height="1326" alt="image" src="https://github.com/user-attachments/assets/1cb0a648-f7fc-4618-b48a-24ad2720c8f6" />

Artifacts = data sharing between jobs

---

### Task 5: Run Real Tests in CI
Take any script from your earlier days (Python or Shell) and run it in CI:
1. Add your script to the `github-actions-practice` repo
2. Write a workflow that:
   - Checks out the code
   - Installs any dependencies needed
   - Runs the script
   - Fails the pipeline if the script exits with a non-zero code
3. Intentionally break the script — verify the pipeline goes red
4. Fix it — verify it goes green again

---

<img width="1704" height="932" alt="image" src="https://github.com/user-attachments/assets/91b4193b-1758-4ac5-939e-358d827ba1e9" />

---

### Task 6: Caching
1. Add `actions/cache` to a workflow that installs dependencies
2. Run it twice — observe the time difference
3. Write in your notes: What is being cached and where is it stored?

---

No difference since the task included in the workflow are not meaningful to be cached.

## artifacts.yaml:

    name: Upload Artifacts
    
    on: push
    
    jobs:
      upload-artifact:
        runs-on: ubuntu-latest
        steps:
    
          - name: Create file
            run: echo "This is my report" > report.txt
    
          - name: Upload Artifact
            uses: actions/upload-artifact@v4
            with:
                name: report
                path: report.txt
      
      job_1:
        runs-on: ubuntu-latest
        steps:
          - name: Create file
            run: echo "Hello from job 1" > file.txt
    
          - name: Upload Artifact
            uses: actions/upload-artifact@v4
            with:
                name: file
                path: file.txt
                
      job_2:
        runs-on: ubuntu-latest
        needs: job_1
        steps:
          - name: Download Artifact
            uses: actions/download-artifact@v4
            with:
                name: file
    
          - name: Read file content
            run: cat file.txt
    
      test:
            runs-on: ubuntu-latest
            
            steps:
                
                - name: Check out code
                  uses: actions/checkout@v4
                
                - name: Cache dependencies
                  uses: actions/cache@v4
                  with:
                     path: ~/.cache
                     key: my-cache-${{ runner.os }}
    
                - name: Install Python
                  run: sudo apt update && sudo apt install -y python3
                
                - name: Run script
                  run: |
                    python3 test.py


## secrets.yaml:

    name: secrets-demo
    
    on: push
    
    jobs:
      test-secrets:
        runs-on: ubuntu-latest
        environment: dev 
    
        steps:
          - name: Check secret exists
            run: |
              if [ -z "${{ secrets.MY_SECRET_MESSAGE }}" ]; then
                echo "Secret is not set"
                exit 1
              else
                echo "Secret is set"
              fi
    
          - name: Try printing secret
            run: echo "${{ secrets.MY_SECRET_MESSAGE }}"
    
          - name: Use secret as env
            env:
                SECRET_MSG: ${{ secrets.MY_SECRET_MESSAGE }}
            run: echo "Using secret safely"

