# Day 30 – Docker Images & Container Lifecycle

## Task
Today's goal is to **understand how images and containers actually work**.

You will:
- Learn the relationship between images and containers
- Understand image layers and caching
- Master the full container lifecycle


---

## Challenge Tasks

### Task 1: Docker Images
1. Pull the `nginx`, `ubuntu`, and `alpine` images from Docker Hub
2. List all images on your machine — note the sizes
3. Compare `ubuntu` vs `alpine` — why is one much smaller?
4. Inspect an image — what information can you see?
5. Remove an image you no longer need

---

<img width="2760" height="1136" alt="image" src="https://github.com/user-attachments/assets/24b179ac-c265-4bd8-9da2-0ded3b4ef735" />

- Pull all the mentioned images and noticed nginx has the hagest disk usage and size

- why alpine is much smaller ?

  -> alpine = minimal Linux
  -> ubuntu = full OS

DID: sudo docker inspect nginx

Saw:
    layers
    config
    environment
    Ports
    Os
    Arch
    Metadata

<img width="1696" height="142" alt="image" src="https://github.com/user-attachments/assets/145cad8b-9e03-42ac-8eb2-62c384fa2eb8" />

---

### Task 2: Image Layers
1. Run `docker image history nginx` — what do you see?
2. Each line is a **layer**. Note how some layers show sizes and some show 0B
3. Write in your notes: What are layers and why does Docker use them?

---

<img width="2254" height="830" alt="image" src="https://github.com/user-attachments/assets/ecce5be1-6be7-46b7-a703-4e94308ca48e" />


saw a complete history when it was first pulled and when it was pulled later layer by layer

  Layers = incremental changes
  Reused -> faster builds
  Saves storage

  Layers created:
    
    Base image → ubuntu
    apt update → new layer
    install nginx → new layer
    copy files → new layer

    Each step = incremental change

 Reuse layers → faster builds
 Example:

  If only COPY changes
     
     Docker reuses previous layers
     builds faster 

---

### Task 3: Container Lifecycle
Practice the full lifecycle on one container:
1. **Create** a container (without starting it)
2. **Start** the container
3. **Pause** it and check status
4. **Unpause** it
5. **Stop** it
6. **Restart** it
7. **Kill** it
8. **Remove** it

Check `docker ps -a` after each step — observe the state changes.

---

<img width="2814" height="880" alt="image" src="https://github.com/user-attachments/assets/2bdd8f6f-33a6-4e41-9d9a-1790cc28a7c1" />

---

### Task 4: Working with Running Containers
1. Run an Nginx container in detached mode
2. View its **logs**
3. View **real-time logs** (follow mode)
4. **Exec** into the container and look around the filesystem
5. Run a single command inside the container without entering it
6. **Inspect** the container — find its IP address, port mappings, and mounts

---

<img width="1518" height="804" alt="image" src="https://github.com/user-attachments/assets/1599ce30-03cb-4bd3-b03b-7d6672b1d612" />

---

### Task 5: Cleanup
1. Stop all running containers in one command
2. Remove all stopped containers in one command
3. Remove unused images
4. Check how much disk space Docker is using

---


<img width="1278" height="208" alt="image" src="https://github.com/user-attachments/assets/6b58272e-6106-406b-a6e7-99d97679a432" />


<img width="1244" height="218" alt="image" src="https://github.com/user-attachments/assets/7ff62746-ceb5-4841-a37f-224100c227d3" />


<img width="1664" height="934" alt="image" src="https://github.com/user-attachments/assets/dfc653c8-4b67-4c0e-941f-d15b3158fc26" />


<img width="1172" height="234" alt="image" src="https://github.com/user-attachments/assets/ac424965-06fd-4600-8dfb-9ae65a043327" />

## What I Learned
- Images are templates for containers
- Containers go through lifecycle states
- Docker uses layers to optimize storage

## Commands Used
docker pull  
docker images  
docker inspect  
docker image history  
docker create  
docker start  
docker stop  
docker pause  
docker unpause  
docker restart  
docker kill  
docker rm  
docker logs  
docker exec  
docker system prune  

## Observations
- Alpine image is very small compared to Ubuntu
- Containers change states during lifecycle
- Layers help in faster builds and reuse

## Container Lifecycle
create → start → pause → unpause → stop → restart → kill → remove


Ciaos Adios :)
