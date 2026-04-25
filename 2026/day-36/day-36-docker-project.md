# Day 36 – Docker Project: Dockerize a Full Application

## Task
Today's goal is to **take a real application and Dockerize it end-to-end**.

No tutorials. No hand-holding. Pick an app, write the Dockerfile, set up Compose, and ship it. This is what you'll do on the job.

---

## Challenge Tasks

### Task 1: Pick Your App
Choose **one** of these (or use your own project):
- A **Python Flask/Django** app with a database
- A **Node.js Express** app with MongoDB
- A **static website** served by Nginx with a backend API
- Any app from your GitHub that doesn't have Docker yet

If you don't have an app, clone a simple open-source one and Dockerize it.

---

Created Node.js + Express API

app.js:

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

package.json:

    {
      "name": "docker-app",
      "version": "1.0.0",
      "main": "app.js",
      "dependencies": {
        "express": "^4.18.2",
        "mongoose": "^7.0.0"
      }
    }

---

### Task 2: Write the Dockerfile
1. Create a Dockerfile for your application
2. Use a **multi-stage build** if applicable
3. Use a **non-root user**
4. Keep the image **small** — use alpine or slim base images
5. Add a `.dockerignore` file

Build and test it locally.

---

Dockerfile:

    # -------- Stage 1: Builder --------
      FROM node:18-alpine AS builder
      
      WORKDIR /app
      
      COPY package.json ./
      RUN npm install
      
      COPY . .

    # -------- Stage 2: Final --------
      FROM node:18-alpine
      
      WORKDIR /app
      
      # Create non-root user
      RUN adduser -D appuser
      
      # Copy only required files from builder
      COPY --from=builder /app /app
      
      # Fix permissions
      RUN chown -R appuser /app
      
      # Switch user
      USER appuser
      
      CMD ["node", "app.js"]


<img width="1012" height="324" alt="image" src="https://github.com/user-attachments/assets/202c1d88-cdd2-4a15-93e7-edb7d1c75f4f" />

---

### Task 3: Add Docker Compose
Write a `docker-compose.yml` that includes:
1. Your **app** service (built from Dockerfile)
2. A **database** service (Postgres, MySQL, MongoDB — whatever your app needs)
3. **Volumes** for database persistence
4. A **custom network**
5. **Environment variables** for configuration (use `.env` file)
6. **Healthchecks** on the database

Run `docker compose up` and verify everything works together.

---

docker-compe.yml:
    
    version: "3.8"
    
    services:
    
      app:
        build: ./app
        ports:
          - "3000:3000"
        environment:
          - MONGO_URI=${MONGO_URI}
        depends_on:
          mongo:
            condition: service_healthy
        networks:
          - app-net
    
      mongo:
        image: mongo
        volumes:
          - mongo_data:/data/db
        networks:
          - app-net
        healthcheck:
          test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
          interval: 5s
          retries: 5
    
    volumes:
      mongo_data:
    
    networks:
      app-net:

.env:

    MONGO_URI=mongodb://mongo:27017/mydb


---

### Task 4: Ship It
1. Tag your app image
2. Push it to Docker Hub
3. Share the Docker Hub link
4. Write a `README.md` in your project with:
   - What the app does
   - How to run it with Docker Compose
   - Any environment variables needed

---

 docker login 
 docker images 
 docker tag app-app warriorr/app-app
 docker push warriorr/app-app

---

### Task 5: Test the Whole Flow
1. Remove all local images and containers
2. Pull from Docker Hub and run using only your compose file
3. Does it work fresh? If not — fix it until it does

---

<img width="1358" height="956" alt="image" src="https://github.com/user-attachments/assets/eea01235-cc45-4d37-bca6-74933d566678" />

---

# Day 36 – Docker Project

## Project
Node.js + MongoDB application

## What I Did
- Created Node.js app
- Dockerized using Dockerfile
- Used non-root user
- Created docker-compose with MongoDB
- Used volumes for persistence
- Added healthcheck and environment variables
- Pushed image to Docker Hub

## Challenges
- Mongo connection failed initially → fixed using service name (mongo)
- App started before DB → fixed using healthcheck

## Final Image Size
~100MB (using alpine)

## Docker Hub Link
https://hub.docker.com/r/warriorr/app-app
