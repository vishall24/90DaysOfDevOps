# Day 40 – Your First GitHub Actions Workflow

## Task
Today you write your **first GitHub Actions pipeline** and watch it run in the cloud.

This is the moment CI/CD stops being a concept and becomes real.

---

## Challenge Tasks

### Task 1: Set Up
1. Create a new **public** GitHub repository called `github-actions-practice`
2. Clone it locally
3. Create the folder structure: `.github/workflows/`

---

---

### Task 2: Hello Workflow
Create `.github/workflows/hello.yml` with a workflow that:
1. Triggers on every `push`
2. Has one job called `greet`
3. Runs on `ubuntu-latest`
4. Has two steps:
   - Step 1: Check out the code using `actions/checkout`
   - Step 2: Print `Hello from GitHub Actions!`

Push it. Go to the **Actions** tab on GitHub and watch it run.

**Verify:** Is it green? Click into the job and read every step.

---

<img width="2406" height="1042" alt="image" src="https://github.com/user-attachments/assets/419d2199-3205-4f2f-b43d-47403f6ab9d1" />

<img width="2406" height="352" alt="image" src="https://github.com/user-attachments/assets/8e4af3a2-a785-4ea2-b755-a30f9496d1f1" />


---

### Task 3: Understand the Anatomy
Look at your workflow file and write in your notes what each key does:
- `on:`
- `jobs:`
- `runs-on:`
- `steps:`
- `uses:`
- `run:`
- `name:` (on a step)

---

## Workflow Keys

    on:
    Defines when pipeline runs (push, PR...)
    
    jobs:
    Group of tasks
    
    runs-on:
    Runner machine (Ubuntu VM)
    
    steps:
    Sequence of actions
    
    uses:
    Prebuilt action (like checkout) built by github 
    
    run:
    Shell command execution
    
    name:
    Label for step


---

### Task 4: Add More Steps
Update `hello.yml` to also:
1. Print the current date and time
2. Print the name of the branch that triggered the run (hint: GitHub provides this as a variable)
3. List the files in the repo
4. Print the runner's operating system

Push again — watch the new run.

---

Failed:

<img width="1952" height="1418" alt="image" src="https://github.com/user-attachments/assets/f31725f0-ef77-4299-8426-90eb0b470072" />

Fixed:

<img width="1616" height="930" alt="image" src="https://github.com/user-attachments/assets/ed6bc551-4b16-4f2d-aed4-ba959d140df2" />

---

### Task 5: Break It On Purpose
1. Add a step that runs a command that will **fail** (e.g., `exit 1` or a misspelled command)
2. Push and observe what happens in the Actions tab
3. Fix it and push again

Write in your notes: What does a failed pipeline look like? How do you read the error?

---

Already done in 4th task.


My Hello.yaml workflow:

    name: First Workflow
    
    on:
      push:
    
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
