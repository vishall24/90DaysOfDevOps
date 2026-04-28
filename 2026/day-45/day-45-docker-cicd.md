# Day 45 – Docker Build & Push in GitHub Actions

## Task
Today you build a **complete CI/CD pipeline** — code pushed to GitHub automatically builds a Docker image and ships it to Docker Hub. No manual steps.

This is exactly what happens in real production pipelines.

---

## Challenge Tasks

### Task 1: Prepare
1. Use the app you Dockerized on Day 36 (or any simple Dockerfile)
2. Add the Dockerfile to your `github-actions-practice` repo (or create a minimal one)
3. Make sure `DOCKER_USERNAME` and `DOCKER_TOKEN` secrets are set from Day 44

---

Docker file is getting build successfully:

<img width="1489" height="799" alt="image" src="https://github.com/user-attachments/assets/33d17d3b-109e-4ec8-b1a4-892b710152f8" />

---

### Task 2: Build the Docker Image in CI
Create `.github/workflows/docker-publish.yml` that:
1. Triggers on push to `main`
2. Checks out the code
3. Builds the Docker image and tags it

**Verify:** Check the build step logs — does the image build successfully?


---

### Task 3: Push to Docker Hub
Add steps to:
1. Log in to Docker Hub using your secrets
2. Tag the image as `username/repo:latest` and also `username/repo:sha-<short-commit-hash>`
3. Push both tags

**Verify:** Go to Docker Hub — is your image there with both tags?

---


<img width="1000" height="569" alt="image" src="https://github.com/user-attachments/assets/8657cbe4-3d3d-42f1-991a-d344178c04ad" />

<img width="834" height="839" alt="image" src="https://github.com/user-attachments/assets/97ff0711-e31f-4883-b2f9-5f731fa3a2d5" />

---

### Task 4: Only Push on Main
Add a condition so the push step only runs on the `main` branch — not on feature branches or PRs.

Test it: push to a feature branch and verify the image is built but NOT pushed.

---

added:

    - name: Push image
      if: github.ref == 'refs/heads/main'
      run: |
        docker push ${{ secrets.DOCKER_USERNAME }}/my-app:latest
        docker push ${{ secrets.DOCKER_USERNAME }}/my-app:sha-${SHA_SHORT}

git checkout -b test-branch

Push

Result:

    Build runs
    Push does NOT happen

---

### Task 5: Add a Status Badge
1. Get the badge URL for your `docker-publish` workflow from the Actions tab
2. Add it to your `README.md`
3. Push — the badge should show green

---

steps to add badge:


<img width="1517" height="633" alt="image" src="https://github.com/user-attachments/assets/046e0f2a-9e05-4e82-9ed1-70a076730bae" />

<img width="790" height="548" alt="image" src="https://github.com/user-attachments/assets/4a48c6e8-26fd-4981-a82c-b2356534a516" />

<img width="1682" height="892" alt="image" src="https://github.com/user-attachments/assets/7db909bd-51f6-4d5d-82a2-cdfb92acf6be" />

Result:

<img width="1093" height="780" alt="image" src="https://github.com/user-attachments/assets/dda8ee8d-5e2a-4381-a4fa-fac25cbc5691" />

---

### Task 6: Pull and Run It
1. On your local machine (or a cloud server), pull the image you just pushed
2. Run it
3. Confirm it works

Write in your notes: What is the full journey from `git push` to a running container?

---

pulled and the application was running.

git push → GitHub triggers workflow → runner starts → code checkout → Docker image builds → login to Docker Hub → image tagged → image pushed → image stored in Docker Hub → user pulls → container runs



## docker-publish.yaml :

    name: Docker CI/CD
    
    on:
      push:
        branches: [main]
    
    jobs:
      build:
        runs-on: ubuntu-latest
        environment: dev
    
        steps:
          - name: Checkout code
            uses: actions/checkout@v4
    
          - name: Login into Docker Hub
            uses: docker/login-action@v3
            with:
                username: ${{ vars.DOCKER_USERNAME }}
                password: ${{ secrets.DOCKER_TOKEN}}
    
          - name: Extract short SHA
            id: vars
            run: echo "SHA_short=${GITHUB_SHA::7}" >> $GITHUB_ENV
            
          - name: BUILD image
            run: |
                docker build -t ${{ vars.DOCKER_USERNAME }}/my-app:latest .
                docker build -t ${{ vars.DOCKER_USERNAME }}/my-app:sha-${SHA_short} .
            
          - name: Push image
            if: github.ref == 'refs/heads/main'
            run: |
                docker push ${{ vars.DOCKER_USERNAME }}/my-app:latest
                docker push ${{ vars.DOCKER_USERNAME }}/my-app:sha-${SHA_short}


## DockerFile:

      FROM node:18-alpine
      
      WORKDIR /app
      
      # Create non-root user
      RUN adduser -D appuser
      
      # Fix permissions
      RUN chown -R appuser /app
      
      # Switch user
      USER appuser
      
      COPY . .
      
      RUN npm install
      
      CMD ["node", "app.js"]


  ## test.py :

     print("Test running")

  ## package.json :

      {
      "name": "docker-app",
      "version": "1.0.0",
      "main": "app.js",
      "dependencies": {
        "express": "^4.18.2",
        "mongoose": "^7.0.0"
      }
    }

  ## app.js :

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
