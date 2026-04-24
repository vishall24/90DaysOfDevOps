# Day 35 – Multi-Stage Builds & Docker Hub

## Task
Today's goal is to **build optimized images and share them with the world**.

Multi-stage builds are how real teams ship small, secure images. Docker Hub is how you distribute them. Both are interview favourites.

---

## Challenge Tasks

### Task 1: The Problem with Large Images
1. Write a simple Go, Java, or Node.js app (even a "Hello World" is fine)
2. Create a Dockerfile that builds and runs it in a **single stage**
3. Build the image and check its **size**

Note down the size — you'll compare it later.

---

The size of my node js application is 1.5GB:
<img width="970" height="291" alt="image" src="https://github.com/user-attachments/assets/f87fc46a-e447-4b21-9007-07609884b58d" />

Dockerfile:

    FROM node:18
    WORKDIR /app
    COPY . .
    RUN npm install
    CMD ["node", "app.js"]

app.js:

    console.log("Hello from Vishal 🚀");

---

### Task 2: Multi-Stage Build
1. Rewrite the Dockerfile using **multi-stage build**:
   - Stage 1: Build the app (install dependencies, compile)
   - Stage 2: Copy only the built artifact into a minimal base image (`alpine`, `distroless`, or `scratch`)
2. Build the image and check its size again
3. Compare the two sizes

Write in your notes: Why is the multi-stage image so much smaller?

---

<img width="971" height="51" alt="image" src="https://github.com/user-attachments/assets/a15a583e-c24f-46ec-8963-5a4ba6aa5cc5" />

The size dropped from 1.5 gb to 181 mb

why? : Multi-stage removes:

        build tools
        cache
        unnecessary dependencies
        
        Only final app is copied

        and only the last image gets considered and all the previous images acts as an cache/layer.

---

### Task 3: Push to Docker Hub
1. Create a free account on [Docker Hub](https://hub.docker.com) (if you don't have one)
2. Log in from your terminal
3. Tag your image properly: `yourusername/image-name:tag`
4. Push it to Docker Hub
5. Pull it on a different machine (or after removing locally) to verify

---

<img width="1408" height="225" alt="image" src="https://github.com/user-attachments/assets/1d3291df-67ef-49a6-8714-9b3812b13c64" />

<img width="1307" height="513" alt="image" src="https://github.com/user-attachments/assets/f4692b89-f94c-4a8e-81ba-ff43a83544e7" />

<img width="1153" height="141" alt="image" src="https://github.com/user-attachments/assets/0d4b0645-0906-4031-a7c3-d2db186ee7e4" />

---

### Task 4: Docker Hub Repository
1. Go to Docker Hub and check your pushed image
2. Add a **description** to the repository
3. Explore the **tags** tab — understand how versioning works
4. Pull a specific tag vs `latest` — what happens?

---

warriorr/my-app:v1
warriorr/my-app:latest

latest = default
v1 = version

---

### Task 5: Image Best Practices
Apply these to one of your images and rebuild:
1. Use a **minimal base image** (alpine vs ubuntu — compare sizes)
2. **Don't run as root** — add a non-root USER in your Dockerfile
3. Combine `RUN` commands to **reduce layers**
4. Use **specific tags** for base images (not `latest`)

Check the size before and after.

---

Docker file with best practice:

    # Use minimal base image (specific version, not latest)
    FROM node:18.17-alpine

    WORKDIR /app
    
    # Create non-root user
    RUN adduser -D appuser
    
    # Copy only package files first (better caching)
    COPY package*.json ./
    
    # Install dependencies
    RUN npm install
    
    # Copy remaining app files
    COPY . .
    
    # Change ownership to non-root user
    RUN chown -R appuser /app
    
    # Switch to non-root user
    USER appuser
    
    EXPOSE 3000
    
    CMD ["node", "app.js"]

<img width="945" height="388" alt="image" src="https://github.com/user-attachments/assets/002f98fc-8107-4273-8661-a0a921599780" />

The size of the docker file is now 237mb since its not multistage + best security.

