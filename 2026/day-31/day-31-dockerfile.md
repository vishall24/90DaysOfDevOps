# Day 31 – Dockerfile: Build Your Own Images

## Task
Today's goal is to **write Dockerfiles and build custom images**.

This is the skill that separates someone who uses Docker from someone who actually ships with Docker.

---

# Challenge Tasks

### Task 1: Your First Dockerfile
1. Create a folder called `my-first-image`
2. Inside it, create a `Dockerfile` that:
   - Uses `ubuntu` as the base image
   - Installs `curl`
   - Sets a default command to print `"Hello from my custom image!"`
3. Build the image and tag it `my-ubuntu:v1`
4. Run a container from your image

**Verify:** The message prints on `docker run`

---


📄 Dockerfile:
    
    FROM ubuntu
    RUN apt update && apt install -y curl
    CMD ["echo", "THIS IS CUSTOM DOCKERFILE"]

Commands:

    docker build -t my-ubuntu:v1 .
    docker run my-ubuntu:v1

---

### Task 2: Dockerfile Instructions
Create a new Dockerfile that uses **all** of these instructions:
- `FROM` — base image
- `RUN` — execute commands during build
- `COPY` — copy files from host to image
- `WORKDIR` — set working directory
- `EXPOSE` — document the port
- `CMD` — default command

Build and run it. Understand what each line does.

---

My answer:

<img width="600" height="559" alt="image" src="https://github.com/user-attachments/assets/8b667095-84c9-4b51-8536-6a135f9740a6" />


Understand:
 
    FROM → base
    RUN → install stuff
    WORKDIR → working dir
    COPY → copy files
    EXPOSE → doc port
    CMD → default command

---

### Task 3: CMD vs ENTRYPOINT
1. Create an image with `CMD ["echo", "hello"]` — run it, then run it with a custom command. What happens?
2. Create an image with `ENTRYPOINT ["echo"]` — run it, then run it with additional arguments. What happens?
3. Write in your notes: When would you use CMD vs ENTRYPOINT?

---

CMD example
 
  FROM ubuntu
  CMD ["echo", "hello"]

Run:

  docker run image-name
  docker run image-name ls   --> list all the files/folders

second overrides Whole CMD ( is it NOT " echo hello ls ", "echo hello" will be replaced by ls )

ENTRYPOINT example
  
  FROM ubuntu
  ENTRYPOINT ["echo"]

Run:
 
  docker run image-name hello
  
  output: hello

CMD → default, can override
ENTRYPOINT → fixed behavior

---

### Task 4: Build a Simple Web App Image
1. Create a small static HTML file (`index.html`) with any content
2. Write a Dockerfile that:
   - Uses `nginx:alpine` as base
   - Copies your `index.html` to the Nginx web directory
3. Build and tag it `my-website:v1`
4. Run it with port mapping and access it in your browser

---

My answer:

<img width="568" height="423" alt="image" src="https://github.com/user-attachments/assets/e130bc6d-1090-43f2-9d4e-8b5fd63ada99" />

---

### Task 5: .dockerignore
1. Create a `.dockerignore` file in one of your project folders
2. Add entries for: `node_modules`, `.git`, `*.md`, `.env`
3. Build the image — verify that ignored files are not included

---

touch .dockerignore
vim .dockerignore:

Add:
  node_modules
  .git
  *.md
  .env

these files will get ignored whie copying the files
reduces image size

---

### Task 6: Build Optimization
1. Build an image, then change one line and rebuild — notice how Docker uses **cache**
2. Reorder your Dockerfile so that frequently changing lines come **last**
3. Write in your notes: Why does layer order matter for build speed?

---

My answer:

First build → slow
Second build → fast

Why?
 
 Docker uses layers cache

  Docker caches layers
  Changing upper layer rebuilds only that
  Order matters → stable first, changing last

Ciao Adios
