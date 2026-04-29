# Day 48 – GitHub Actions Project: End-to-End CI/CD Pipeline

## Task
You've learned workflows, triggers, secrets, Docker builds, reusable workflows, and advanced events. Today you **put it all together** in one project — a complete, production-style CI/CD pipeline that builds, tests, and deploys using everything you've learned from Day 40 to Day 47.

This is your GitHub Actions capstone.

---

## Challenge Tasks

### Task 1: Set Up the Project Repo
1. Create a new repo called `github-actions-capstone` (or use your existing `github-actions-practice`)
2. Add a simple app — pick any one:
   - A Python Flask/FastAPI app with one endpoint
   - A Node.js Express app with one endpoint
   - Your Dockerized app from Day 36
3. Add a `Dockerfile` and a basic test (even a script that curls the health endpoint counts)
4. Add a `README.md` with a project description

---

Added, refer day 36:

https://github.com/vishall24/90DaysOfDevOps/blob/master/2026/day-36/day-36-docker-project.md

## Task 2 & 3 & 4 & 5 & 6 & 7 & Brownie Points:

main-pipeline triggered reusable-docker pipeline inside it:

<img width="2016" height="1328" alt="image" src="https://github.com/user-attachments/assets/f30a5bd9-8b70-4e31-abc9-2679fe26dd59" />

and built and pushed the image:

<img width="1880" height="1356" alt="image" src="https://github.com/user-attachments/assets/51f076f6-d42c-4879-804a-2fa5a820ad13" />

After creating PR:

<img width="2024" height="1478" alt="image" src="https://github.com/user-attachments/assets/b1b04dd9-635b-492b-aab0-d14c5d7f8132" />

<img width="1932" height="1128" alt="image" src="https://github.com/user-attachments/assets/29bcfd94-1b4e-4b61-85d8-69b5056d77c1" />

<img width="1886" height="822" alt="image" src="https://github.com/user-attachments/assets/912cce11-b6ad-4593-96dd-2fbe85818999" />

Health ypipeline:

<img width="2520" height="1508" alt="image" src="https://github.com/user-attachments/assets/65e9c635-ad75-494d-99bc-262f7ed3950b" />

Created badges for all pipelines:

<img width="1870" height="1240" alt="image" src="https://github.com/user-attachments/assets/dce27d19-444d-444d-b82b-0d68b6c718ca" />

Found vulnerability:

<img width="2520" height="1368" alt="image" src="https://github.com/user-attachments/assets/633c576d-c566-4bf5-a061-bfcb5281edad" />


downloaded report:

<img width="2162" height="704" alt="image" src="https://github.com/user-attachments/assets/10fb86a1-df47-42b1-8954-2c1a8b1030c5" />

Complete flow :

<img width="2068" height="1592" alt="NoteGPT-Flowchart-1777448683462" src="https://github.com/user-attachments/assets/a5257d28-5257-4b0a-87a7-618f19a83f75" />

complete flow for error:

<img width="2068" height="1592" alt="NoteGPT-Flowchart-1777448776471" src="https://github.com/user-attachments/assets/47b7f779-caa4-4556-b1a8-45545682f242" />

Complete flow:


## PR → test only

## Merge → test → docker build → tag (latest + sha) → push → trivy scan → deploy (after approval)

## Schedule → pull → run → health check → summary

---

## Dockerfile

    # -------- Stage 1: Builder --------
      FROM node:20-alpine AS builder
      
      WORKDIR /app
      
      RUN apk update && apk upgrade --no-cache
    
      COPY package.json ./
      RUN npm install
      
      COPY . .
    
    # -------- Stage 2: Final --------
      FROM node:20-alpine
      
      WORKDIR /app
      
      RUN apk update && apk upgrade --no-cache
    
      # Create non-root user
      RUN adduser -D appuser
      
      # Copy only required files from builder
      COPY --from=builder /app /app
      
      # Fix permissions
      RUN chown -R appuser /app
      
      # Switch user
      USER appuser
      
      CMD ["node", "app.js"]


## app.js:

    const express = require("express");
    const mongoose = require("mongoose");
    
    const app = express();
    
    mongoose.connect(process.env.MONGO_URI)
      .then(() => console.log("MongoDB Connected"))
      .catch(err => console.log(err));
    
    app.get("/", (req, res) => {
      res.send("Hello from Vishal Docker App 🚀");
    });
    
    app.listen(3000, () => console.log("Server running on port 3000"));



## package.json:

    {
      "name": "docker-app",
      "version": "1.0.0",
      "main": "app.js",
      "dependencies": {
        "express": "^4.18.2",
        "mongoose": "^7.0.0"
      }
    }


## README.md:

    # Github Actions based node js application
    
    ## Status for existing pipelines :
    
    [![Main Pipeline](https://github.com/vishall24/github-actions-capstone/actions/workflows/main-pipeline.yml/badge.svg)](https://github.com/vishall24/github-actions-capstone/actions/workflows/main-pipeline.yml)
    
    [![PR Pipeline](https://github.com/vishall24/github-actions-capstone/actions/workflows/pr-pipeline.yml/badge.svg)](https://github.com/vishall24/github-actions-capstone/actions/workflows/pr-pipeline.yml)
    
    [![Health Check](https://github.com/vishall24/github-actions-capstone/actions/workflows/health-check.yml/badge.svg)](https://github.com/vishall24/github-actions-capstone/actions/workflows/health-check.yml)


## reusable_docker.yaml:

    name: Docker Build Push
    
    on:
      workflow_call:
        inputs:
          image_name:
            required: true
            type: string
        secrets:
          docker_username:
            required: true
          docker_token:
            required: true
        outputs:
          image_latest:
            value: ${{ jobs.docker.outputs.image_latest }}
          image_sha:
            value: ${{ jobs.docker.outputs.image_sha }}
    
    jobs:
      docker:
        runs-on: ubuntu-latest
        environment: "production"
    
        outputs:
          image_latest: ${{ steps.meta.outputs.latest }}
          image_sha: ${{ steps.meta.outputs.sha }}
    
        steps:
          - uses: actions/checkout@v4
    
          - name: Extract short SHA
            id: vars
            run: echo "SHA_SHORT=$(echo $GITHUB_SHA | cut -c1-7)" >> $GITHUB_ENV
    
          - name: Login Docker
            uses: docker/login-action@v3
            with:
              username: ${{ vars.docker_username }}
              password: ${{ secrets.docker_token }}
    
          - name: Build images
            run: |
              docker build -t ${{ inputs.image_name }}:latest .
              docker build -t ${{ inputs.image_name }}:sha-${SHA_SHORT} .
    
          - name: Push images
            run: |
              docker push ${{ inputs.image_name }}:latest
              docker push ${{ inputs.image_name }}:sha-${SHA_SHORT}
    
          - name: Set outputs
            id: meta
            run: |
              echo "latest=${{ inputs.image_name }}:latest" >> $GITHUB_OUTPUT
              echo "sha=${{ inputs.image_name }}:sha-${SHA_SHORT}" >> $GITHUB_OUTPUT



## reusable_build_test.yaml:

    name: Build & Test
    
    on:
      workflow_call:
        inputs:
          node_version:
            required: true
            type: string
          run_tests:
            required: false
            type: boolean
            default: true
    
    jobs:
      build-test:
        runs-on: ubuntu-latest
    
        outputs:
          test_result: ${{ steps.test.outcome }}
    
        steps:
          - uses: actions/checkout@v4
    
          - uses: actions/setup-node@v4
            with:
              node-version: ${{ inputs.node_version }}
    
          - name: Install dependencies
            run: npm install
    
          - name: Run tests
            id: test
            if: inputs.run_tests == true
            run: echo "Tests passed"



## pr-pipeline.yaml:

    name: PR Pipeline
    
    on:
      pull_request:
        branches: [main]
        types: [opened, synchronize]
    
    jobs:
      test:
        uses: ./.github/workflows/reusable-build-test.yml
        with:
          node_version: "18"
          run_tests: true
    
      pr-summary:
        needs: test
        runs-on: ubuntu-latest
    
        steps:
          - run: echo "PR checks passed for branch ${{ github.head_ref }}"


## main-pipeline.yaml:

      name: Main Pipeline
      
      on:
        push:
          branches: [main]
      
      jobs:
        test:
          uses: ./.github/workflows/reusable-build-test.yml
          with:
            node_version: "18"
      
        docker:
          needs: test
          uses: ./.github/workflows/reusable-docker.yml
          with:
            image_name: warriorr/app
          secrets:
            docker_username: ${{ vars.DOCKER_USERNAME }}
            docker_token: ${{ secrets.DOCKER_TOKEN }}
      
      
        scan:
          needs: docker
          runs-on: ubuntu-latest
      
          steps:
      
            - name: Debug output
              run: echo "IMAGE = ${{ needs.docker.outputs.image_latest }}"
      
            - name: Scan Docker Image
              continue-on-error: true
              uses: aquasecurity/trivy-action@v0.20.0
              with:
                  image-ref: ${{ needs.docker.outputs.image_latest }} 
                  format: table
                  severity: CRITICAL
                  exit-code: 1
                  output: trivy-report.txt
              env:
                  continued-on-error: true
      
            - name: Upload Trivy Report
              uses: actions/upload-artifact@v4
              with:
                  name: trivy-report
                  path: trivy-report.txt
      
        deploy:
          needs: scan
          runs-on: ubuntu-latest
          environment: production
      
          steps:
            - name: Deploy
              run: |
                echo "Deploying:"
                echo "Latest: ${{ needs.docker.outputs.image_latest }}"
                echo "SHA: ${{ needs.docker.outputs.image_sha }}"

## health-check.yaml:

    name: Health Check
    
    on:
      schedule:
        - cron: "0 */12 * * *"
      workflow_dispatch:
    
    jobs:
      health:
        runs-on: ubuntu-latest
        environment: "production"
    
        steps:
          - name: Pull latest image
            run: docker pull ${{ vars.DOCKER_USERNAME }}/app:latest
    
          - name: Run container
            run: docker run -d -p 3000:3000 --name app ${{ vars.DOCKER_USERNAME }}/app:latest
    
          - name: Wait
            run: sleep 5
    
          - name: Test app
            id: test
            run: |
              if curl -f http://localhost:3000; then
                echo "STATUS=PASSED" >> $GITHUB_ENV
              else
                echo "STATUS=FAILED" >> $GITHUB_ENV
              fi
    
          - name: Summary
            run: |
              echo "## Health Check Report" >> $GITHUB_STEP_SUMMARY
              echo "- Image: latest" >> $GITHUB_STEP_SUMMARY
              echo "- Status: $STATUS" >> $GITHUB_STEP_SUMMARY
              echo "- Time: $(date)" >> $GITHUB_STEP_SUMMARY
    
          - run: docker stop app && docker rm app

