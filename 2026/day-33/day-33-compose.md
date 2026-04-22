# Day 33 – Docker Compose: Multi-Container Basics

## Task
Today's goal is to **run multi-container applications with a single command**.

Yesterday you manually created networks and volumes and ran containers one by one. Docker Compose does all of that in one YAML file.

---

## Challenge Tasks

### Task 1: Install & Verify
1. Check if Docker Compose is available on your machine
2. Verify the version

---

My answer:

docker compose version
= Docker Compose version v5.1.3

---

### Task 2: Your First Compose File
1. Create a folder `compose-basics`
2. Write a `docker-compose.yml` that runs a single **Nginx** container with port mapping
3. Start it with `docker compose up`
4. Access it in your browser
5. Stop it with `docker compose down`

---

My answer:

<img width="1772" height="118" alt="image" src="https://github.com/user-attachments/assets/e9bfc5bc-4609-427a-9401-e762e7c35821" />

docker-compose.yml:
  
    services:
      nginx:
        image: nginx
        ports:
          - "8080:80"

<img width="1386" height="501" alt="image" src="https://github.com/user-attachments/assets/b282cb66-7e96-4123-a9d6-514a80da2fed" />

<img width="1799" height="109" alt="image" src="https://github.com/user-attachments/assets/122bbb78-4967-478c-8d41-d57b2ea19d3c" />

---

### Task 3: Two-Container Setup
Write a `docker-compose.yml` that runs:
- A **WordPress** container
- A **MySQL** container

They should:
- Be on the same network (Compose does this automatically)
- MySQL should have a named volume for data persistence
- WordPress should connect to MySQL using the service name

Start it, access WordPress in your browser, and set it up.

**Verify:** Stop and restart with `docker compose down` and `docker compose up` — is your WordPress data still there?

---

My answer:

docker compose file:

    version: "3.8"
    services:
      db:
        image: mysql:5.7
        container_name: mysql-db
        restart: always
        environment:
          MYSQL_ROOT_PASSWORD: rootpass
          MYSQL_DATABASE: wordpress
          MYSQL_USER: wpuser
          MYSQL_PASSWORD: wppass
        volumes:
          - db_data:/var/lib/mysql
    
      wordpress:
        image: wordpress:latest
        container_name: wordpress-app
        restart: always
        ports:
          - "8081:80"
        environment:
          WORDPRESS_DB_HOST: db
          WORDPRESS_DB_USER: wpuser
          WORDPRESS_DB_PASSWORD: wppass
          WORDPRESS_DB_NAME: wordpress
        depends_on:
          - db
    
    volumes:
      db_data:
      
Wordpress application UI:
<img width="1902" height="926" alt="image" src="https://github.com/user-attachments/assets/143b4d03-225e-46f8-9cf0-17b6eb87bb79" />

After restarting docker compose , data still there:

<img width="1858" height="971" alt="image" src="https://github.com/user-attachments/assets/806172cb-91e5-4e82-ad6f-f1c77249433a" />

---

### Task 4: Compose Commands
Practice and document these:
1. Start services in **detached mode**
2. View running services
3. View **logs** of all services
4. View logs of a **specific** service
5. **Stop** services without removing
6. **Remove** everything (containers, networks)
7. **Rebuild** images if you make a change

---

Detached mode

    docker compose up -d

Running services

    docker compose ps
    
Logs (all)

    docker compose logs -f

Logs (specific)

    docker compose logs -f wordpress

Stop only

    docker compose stop

Remove everything
    
    docker compose down

Rebuild
    
    docker compose up -d --build

---

### Task 5: Environment Variables
1. Add environment variables directly in your `docker-compose.yml`
2. Create a `.env` file and reference variables from it in your compose file
3. Verify the variables are being picked up

---

My answer:

vim .env:

<img width="496" height="97" alt="image" src="https://github.com/user-attachments/assets/4278a24e-7281-4fc3-9eb1-0dd155bae576" />

<img width="654" height="676" alt="image" src="https://github.com/user-attachments/assets/5733f1a1-4f5f-4280-8cbf-8ae2e62e5e49" />

