# Day 29 – Introduction to Docker

## Task
Today's goal is to **understand what Docker is and run your first container**.

You will:
- Learn why containers exist and how they differ from VMs
- Install Docker on your machine
- Run and explore containers from Docker Hub

---

## Challenge Tasks

### Task 1: What is Docker?
Research and write short notes on:
- What is a container and why do we need them?
- Containers vs Virtual Machines — what's the real difference?
- What is the Docker architecture? (daemon, client, images, containers, registry)

Draw or describe the Docker architecture in your own words.

---

1. What is a container & why we need it?

   A container packages an app with all its dependencies so it runs consistently anywhere.
   It solves “works on my machine” issues and is lightweight and portable.

2. Containers vs Virtual Machines

   Containers share the host OS and are lightweight and fast.
   VMs have a full OS, are heavier, and slower but provide stronger isolation.

3. Docker Architecture

   Docker has client (CLI), daemon (engine), images (templates), containers (running apps), and registry (stores images).
   Flow: client → daemon → image → container.
   
<img width="1000" height="382" alt="image" src="https://github.com/user-attachments/assets/bb328eaa-4725-4988-9113-3724e11287d5" />

   Docker Architecture :

     Client → sends command
     Docker Daemon → runs containers
     Images → blueprint
     Containers → running app
     Registry (Docker Hub) → where images come from

---

### Task 2: Install Docker
1. Install Docker on your machine (or use a cloud instance)
2. Verify the installation
3. Run the `hello-world` container
4. Read the output carefully — it explains what just happened

---

<img width="1610" height="910" alt="image" src="https://github.com/user-attachments/assets/e679c106-baca-4c66-aaee-f1fd27c566c4" />

I ran docker image named : "hello-world" which was present in the registry dockerhub, since the image was not present locally
Docker Daemon (dockerd) will pull the docker image from the registry, then the Docker Daemon runs the container.

---

### Task 3: Run Real Containers
1. Run an **Nginx** container and access it in your browser
2. Run an **Ubuntu** container in interactive mode — explore it like a mini Linux machine
3. List all running containers
4. List all containers (including stopped ones)
5. Stop and remove a container

---

<img width="2818" height="768" alt="image" src="https://github.com/user-attachments/assets/ef894464-0cda-4e02-aeb6-39547cb9220e" />

<img width="2838" height="1334" alt="image" src="https://github.com/user-attachments/assets/7844da33-7e61-4b38-9257-a64e24341f0d" />

---

### Task 4: Explore
1. Run a container in **detached mode** — what's different?
2. Give a container a custom **name**
3. Map a **port** from the container to your host
4. Check **logs** of a running container
5. Run a command **inside** a running container

---

<img width="2818" height="1202" alt="image" src="https://github.com/user-attachments/assets/54b88f5b-328f-4116-a10f-019d30bfe2bf" />

<img width="2152" height="736" alt="image" src="https://github.com/user-attachments/assets/d3d632c7-67c6-4aae-8d6d-5fd326e4163d" />


Ciaos Adios :)
