# Day 49 – DevSecOps: Add Security to Your CI/CD Pipeline

## Task
You can build and deploy automatically. But what if your Docker image has a known vulnerability? What if someone accidentally commits a password? Today you learn **DevSecOps** — adding simple, automated security checks to your pipeline so problems are caught **before** they reach production.

Don't worry — this isn't a security course. You're just adding a few smart steps to the pipeline you already built.

---

1. What is DevSecOps

→ security inside CI/CD pipeline

2. What you implemented
   
Trivy scan
Dependency scan
Secret scanning
Permissions

3. Trivy Output:
   
<img width="2100" height="758" alt="image" src="https://github.com/user-attachments/assets/20e12c35-248c-45a4-b40e-c70163a8341d" />


4. Diagram
   
PR → test → scan

main → build → scan → push → deploy

## Task 1:

Done in day 48

## Task 2:

<img width="2176" height="504" alt="image" src="https://github.com/user-attachments/assets/6745ac2c-c99e-4939-a7d6-778890b5ccd4" />

## Task 3:

<img width="2280" height="884" alt="image" src="https://github.com/user-attachments/assets/c3ecddae-fde3-46ef-9598-3a426bd5a8d4" />

<img width="2164" height="1234" alt="image" src="https://github.com/user-attachments/assets/3172570e-4884-4cfa-bee4-c86a1990a2a0" />

Secret scanning → detects after push
Push protection → blocks before push

If someone pushes:

    AWS_SECRET=abcd123

GitHub will:

    detect it
    block push OR alert

## Task 4:

<img width="1522" height="568" alt="image" src="https://github.com/user-attachments/assets/43b06eb5-155f-4bda-87a7-224dab49faf6" />

    Default GitHub Actions:
    has too much access
    
    If compromised:
    attacker can modify repo

## Task 5:

PR Flow (Before Merge):

    Developer opens PR
       ↓
    Build & Test
       ↓
    Dependency Vulnerability Scan
       ↓
    PASS → PR can be merged
    FAIL → Fix issues

Main Branch Flow (After Merge):

    Code merged to main
       ↓
    Build & Test
       ↓
    Docker Build
       ↓
    Trivy Image Scan (CRITICAL/HIGH)
       ↓
    PASS → Docker Push
    FAIL → Stop pipeline
       ↓
    Deploy (after approval)
    why using permissions?:


Always Active (Background Security):

    GitHub Secret Scanning
       ↓
    Detect leaked secrets in repo
    
    Push Protection
       ↓
    Block commits containing secrets

Brownie points:

1)

Instead of:

    uses: actions/checkout@v4

Use:

    uses: actions/checkout@<commit-sha>
 
= Prevents supply chain attack



<img width="1304" height="698" alt="image" src="https://github.com/user-attachments/assets/86640939-37b0-4e93-a227-fe1215ab5cb3" />


2)

<img width="2834" height="1450" alt="image" src="https://github.com/user-attachments/assets/659f61a4-378b-4c3a-aea9-9878dc8f2206" />
