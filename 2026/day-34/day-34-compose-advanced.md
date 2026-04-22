# Day 34 – Docker Compose: Real-World Multi-Container Apps

## Task
Today's goal is to **build more complex, production-like setups with Docker Compose**.

Yesterday was basics. Today you handle real scenarios — app + database + cache, healthchecks, restart policies, and service dependencies.

---

## Challenge Tasks

### Task 1: Build Your Own App Stack
Create a `docker-compose.yml` for a 3-service stack:
- A **web app** (use Python Flask, Node.js, or any language you know)
- A **database** (Postgres or MySQL)
- A **cache** (Redis)

Write a simple Dockerfile for the web app. The app doesn't need to be complex — even a "Hello World" that connects to the database is enough.

---

Application runing:
<img width="1378" height="280" alt="image" src="https://github.com/user-attachments/assets/ca06bb3b-3226-4c51-ae6d-cd47a10df7d8" />

Docker compose file:

    version: "3.8"
    
    services:
        db:
          image: postgres
          restart: always
          environment: 
              POSTGRES_USER: user
              POSTGRES_PASSWORD: pass
              POSTGRES_DB: mydb
          volumes:
            - db_data:/var/lib/postgresql
          networks:
            - app-net
          healthcheck:
            test: ["CMD-SHELL", "pg_isready -U user"]
            interval: 5s
            timeout: 5s
            retries: 5
    
        redis:
            image: redis
            networks:
              - app-net
    
        web:
          build: ../app
          ports:
            - "5000:5000"
          depends_on: 
            db:
              condition: service_healthy
          networks:
            - app-net
    
    volumes: 
      db_data:
    
    networks:
      app-net:  


Dockerfile:

    FROM python:3.9-slim
    
    WORKDIR /app
    
    COPY . .
    
    RUN pip install -r requirements.txt
    
    CMD ["python", "app.py"]

app.py:

    from flask import Flask
    import psycopg2
    import redis
    import time
    
    app = Flask(__name__)
    
    @app.route("/")
    def home():
        return "Hello from Vishal 🚀"
    
    if __name__ == "__main__":
        app.run(host="0.0.0.0", port=5000)


---

### Task 2: depends_on & Healthchecks
1. Add `depends_on` to your compose file so the app starts **after** the database
2. Add a **healthcheck** on the database service
3. Use `depends_on` with `condition: service_healthy` so the app waits for the database to be truly ready, not just started

**Test:** Bring everything down and up — does the app wait for the DB?

---

Added healthcheck and ensures that the web service runs only after db using depends_on , check docker compose at task 1

---

### Task 3: Restart Policies
1. Add `restart: always` to your database service
2. Manually kill the database container — does it come back?
3. Try `restart: on-failure` — how is it different?
4. Write in your notes: When would you use each restart policy?

---

Added restart: always

Test:
docker ps
docker kill <db-container-id>

It will not restart automatically. Since it will only restart if any error occured .

always: Restarts the container/service regardless of its exit status (even if manually stopped or exited normally).
on-failure: Restarts only if the container/service exits with a non-zero exit code, indicating a crash or error.

---

### Task 4: Custom Dockerfiles in Compose
1. Instead of using a pre-built image for your app, use `build:` in your compose file to build from a Dockerfile
2. Make a code change in your app
3. Rebuild and restart with one command

---

Already used:

build: ./app


---

### Task 5: Named Networks & Volumes
1. Define **explicit networks** in your compose file instead of relying on the default
2. Define **named volumes** for database data
3. Add **labels** to your services for better organization

---

Already added:

networks:
  app-net:

volumes:
  db_data:


 DB data persists
 Containers talk via name

---

### Task 6: Scaling (Bonus)
1. Try scaling your web app to 3 replicas using `docker compose up --scale`
2. What happens? What breaks?
3. Write in your notes: Why doesn't simple scaling work with port mapping?

---

docker compose up --scale web=3

 Problem:

 Port conflict (5000 already used)

Scaling fails because:
multiple containers cannot bind same host port

# Day 34 – Advanced Compose

## What I Learned
- Multi-container apps using Docker Compose
- depends_on with healthcheck ensures proper startup
- Restart policies manage container failures
- Custom Dockerfile used for app
- Networks enable service communication
- Volumes persist database data

## Observations
- App waits for DB due to healthcheck
- DB restarts automatically
- Scaling fails due to port conflict
