# Day 42 – Runners: GitHub-Hosted & Self-Hosted

## Task
Every job needs a machine to run on. Today you understand **runners** — GitHub's hosted ones and how to set up your own self-hosted runner on a real server.

---

## Challenge Tasks

### Task 1: GitHub-Hosted Runners
1. Create a workflow with 3 jobs, each on a different OS:
   - `ubuntu-latest`
   - `windows-latest`
   - `macos-latest`
2. In each job, print:
   - The OS name
   - The runner's hostname
   - The current user running the job
3. Watch all 3 run in parallel

Write in your notes: What is a GitHub-hosted runner? Who manages it?

---

All jobs were running in parallel:

<img width="979" height="666" alt="image" src="https://github.com/user-attachments/assets/eb9312eb-6e93-4d00-871c-bded9184fcb5" />

hosted.yaml:

      name: Hosted Runners
      
      on:
        push:
      
      
      jobs:
        ubuntu-job:
          runs-on: ubuntu-latest
      
          steps:
      
            - run: |
                echo "OS: ubuntu"
                hostname
                whoami
      
        windows-job:
          runs-on: windows-latest
      
          steps:
      
            - run: |
                echo "OS: Windows"
                hostname
                whoami
      
        mac-job:
          runs-on: macos-latest
      
          steps:
      
            - run: |
                echo "OS: macos"
                hostname
                whoami

- GitHub-hosted runner = temporary VM provided by GitHub
- Managed by GitHub (you don’t control infra)

---

### Task 2: Explore What's Pre-installed
1. On the `ubuntu-latest` runner, run a step that prints:
   - Docker version
   - Python version
   - Node version
   - Git version
2. Look up the GitHub docs for the full list of pre-installed software on `ubuntu-latest`

Write in your notes: Why does it matter that runners come with tools pre-installed?

---

All the dependencies were installed:

<img width="780" height="281" alt="image" src="https://github.com/user-attachments/assets/50102060-1f75-4515-bd5f-c15b224366ba" />

Saves setup time → faster pipelines

---

### Task 3: Set Up a Self-Hosted Runner
1. Go to your GitHub repo → Settings → Actions → Runners → **New self-hosted runner**
2. Choose Linux as the OS
3. Follow the instructions to download and configure the runner on:
   - Your local machine, OR
   - A cloud VM (EC2, Utho, or any VPS)
4. Start the runner — verify it shows as **Idle** in GitHub

**Verify:** Your runner appears in the Runners list with a green dot.

---

Made my local machine as runner:

<img width="1689" height="446" alt="image" src="https://github.com/user-attachments/assets/36c7647c-c4d7-4764-abbf-94347930c020" />

its a windows runner.

---

### Task 4: Use Your Self-Hosted Runner
1. Create `.github/workflows/self-hosted.yml`
2. Set `runs-on: self-hosted`
3. Add steps that:
   - Print the hostname of the machine (it should be YOUR machine/VM)
   - Print the working directory
   - Create a file and verify it exists on your machine after the run
4. Trigger it and watch it run on your own hardware

**Verify:** Check your machine — is the file there?

---

test.txt file at runner:

<img width="1494" height="544" alt="image" src="https://github.com/user-attachments/assets/3da496d2-eb6a-4aed-b67f-8f533acb1333" />

<img width="1403" height="920" alt="image" src="https://github.com/user-attachments/assets/412960b8-d8b3-469d-a66b-8bce81b36ee1" />

---

### Task 5: Labels
1. Add a **label** to your self-hosted runner (e.g., `my-linux-runner`)
2. Update your workflow to use `runs-on: [self-hosted, my-linux-runner]`
3. Trigger it — does it still pick up the job?

Write in your notes: Why are labels useful when you have multiple self-hosted runners?

---

Yes it does pck the runner 

If you have multiple machines → control where job runs and whenever any machine is idle it will pick any machine from pool of runner.

---

### Task 6: GitHub-Hosted vs Self-Hosted
Fill this in your notes:

| | GitHub-Hosted | Self-Hosted |
|---|---|---|
| Who manages it? | Github | us |
| Cost | free(limited) | our infra cost |
| Pre-installed tools | pre-installed | you have to install |
| Good for | Quick CI-CD/testing | custom workloads + Security |
| Security concern | shared environment(No security) | Full control (Full security) |





## Files:

self-hosted.yaml:

    name: Self hosted
    
    on:
      push:
    
    jobs:
      self-jobs:
        runs-on: [self-hosted, my-windows-runner]
    
        steps:
    
        - name: Show hostname
          run: hostname
    
        - name: Show working dir 
          run: pwd
    
        - name: Create files
          run: |
            echo "Hello from github actions to self hosted runner" > test.txt


hosted.yaml:

      name: Hosted Runners
      
      on:
        push:
      
      
      jobs:
        ubuntu-job:
          runs-on: ubuntu-latest
      
          steps:
      
            - run: |
                echo "OS: ubuntu"
                hostname
                whoami
      
            - name: Check tools
              run: |
                docker --version
                python --version
                node --version
                git --version
      
        windows-job:
          runs-on: windows-latest
      
          steps:
      
            - run: |
                echo "OS: Windows"
                hostname
                whoami
      
        mac-job:
          runs-on: macos-latest
      
          steps:
      
            - run: |
                echo "OS: macos"
                hostname
                whoami

                
