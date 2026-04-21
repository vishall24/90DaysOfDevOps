# Day 32 – Docker Volumes & Networking

## Task
Today's goal is to **solve two real problems: data persistence and container communication**.

Containers are ephemeral — they lose data when removed. And by default, containers can't easily talk to each other. Today you fix both.

---

## Challenge Tasks

### Task 1: The Problem
1. Run a Postgres or MySQL container
2. Create some data inside it (a table, a few rows — anything)
3. Stop and remove the container
4. Run a new one — is your data still there?

Write what happened and why.

---


<img width="1716" height="1434" alt="image" src="https://github.com/user-attachments/assets/38e32893-819c-43f8-9e42-f8c24e3cd496" />

---

:: created table and its data

---

<img width="2718" height="1300" alt="image" src="https://github.com/user-attachments/assets/ae89e19a-697f-49ab-8d4d-248c41f6d08a" />

---

:: after re-creating the same image all data lost since the data was not persistent

---

Containers are ephemeral → data stored inside container is lost


---

### Task 2: Named Volumes
1. Create a named volume
2. Run the same database container, but this time **attach the volume** to it
3. Add some data, stop and remove the container
4. Run a brand new container with the **same volume**
5. Is the data still there?

**Verify:** `docker volume ls`, `docker volume inspect`

---

My answer:


<img width="2844" height="226" alt="image" src="https://github.com/user-attachments/assets/2bacfc23-d910-46e0-abf7-3cdb29174de5" />

Data still there:: even after re-creating the same container but with mounted path

<img width="2848" height="978" alt="image" src="https://github.com/user-attachments/assets/201091a2-2565-4ade-ac1a-ef441c486769" />

<img width="1764" height="716" alt="image" src="https://github.com/user-attachments/assets/e656e7ac-6434-4304-af7e-e24c8553ec6f" />

Here :

 Container says:

“I store data in /var/lib/postgresql”

 Docker says:

“I actually store that data here on host:
 /var/lib/docker/volumes/my-db-data/_data” --> this is the place where the data is getting stored on the host machine ( not the container)

 ---

### Task 3: Bind Mounts
1. Create a folder on your host machine with an `index.html` file
2. Run an Nginx container and **bind mount** your folder to the Nginx web directory
3. Access the page in your browser
4. Edit the `index.html` on your host — refresh the browser

Write in your notes: What is the difference between a named volume and a bind mount?

---

My answer:


<img width="2274" height="1152" alt="image" src="https://github.com/user-attachments/assets/6f55d69c-41b8-42a6-bcf0-3a51dbb58b06" />

OUTPUT:

<img width="1596" height="1010" alt="image" src="https://github.com/user-attachments/assets/ba569a91-8a10-495b-97c1-86581879c554" />

DID: vim index.html , added - part 2 at last saved , and changes are here:

<img width="1370" height="594" alt="image" src="https://github.com/user-attachments/assets/4cf4ac38-c815-4a62-aef1-3703b6afd5db" />

Difference
 
  Named volume → managed by Docker
  Bind mount → direct host folder

Bind mount:
    
    dev work
    live editing

Volume:
    
    databases
    persistent storage

---

### Task 4: Docker Networking Basics
1. List all Docker networks on your machine
2. Inspect the default `bridge` network
3. Run two containers on the default bridge — can they ping each other by **name**?
4. Run two containers on the default bridge — can they ping each other by **IP**?

---

My answer:


<img width="1146" height="226" alt="image" src="https://github.com/user-attachments/assets/251a46c0-cfc2-445a-984e-1c5908ece3b3" />

sudo docker network inspect bridge:

<img width="2062" height="1050" alt="image" src="https://github.com/user-attachments/assets/a26f6f37-fde2-4c4f-b1ea-b66e165b94ce" />

sudo docker -dit --name c1 ubuntu
sudo docker -dit --name c2 ubuntu

Tried pinging:

<img width="2714" height="100" alt="image" src="https://github.com/user-attachments/assets/ef02f582-0394-433f-ba78-077213329c96" />
 ping is not installed lets install
 After installation:
 
 <img width="1560" height="936" alt="image" src="https://github.com/user-attachments/assets/f843b079-2b9f-41b5-8338-ab685b4729dd" />

 With IP works ( ping ) , but with container name it doesnt work.

---

### Task 5: Custom Networks
1. Create a custom bridge network called `my-app-net`
2. Run two containers on `my-app-net`
3. Can they ping each other by **name** now?
4. Write in your notes: Why does custom networking allow name-based communication but the default bridge doesn't?

---

After using custom network able to ping with container name:

<img width="1710" height="958" alt="image" src="https://github.com/user-attachments/assets/12cb6454-96fa-4b00-b5fd-79a9eae7c28b" />

commands used:

<img width="1546" height="488" alt="image" src="https://github.com/user-attachments/assets/8877fe89-1d20-4978-9e83-101e59cbb54b" />

Default bridge → IP works, name doesn’t
Custom network → both work

because = Custom Docker networks provide built-in DNS resolution, while the default bridge does not.

---

### Task 6: Put It Together
1. Create a custom network
2. Run a **database container** (MySQL/Postgres) on that network with a volume for data
3. Run an **app container** (use any image) on the same network
4. Verify the app container can reach the database by container name

---

Able to ping from the ubuntu container to postgres container named " db "
<img width="1658" height="574" alt="image" src="https://github.com/user-attachments/assets/b45cf59e-ae66-42c2-b51b-9cdfe189b68c" />

command used:

<img width="2562" height="226" alt="image" src="https://github.com/user-attachments/assets/f09bca46-13db-4d9d-92fa-1345d77fe078" />

we could have used bridge/default network as well but in that case we would be ping the other service using its IP not by its name

Ciao Adios :)
