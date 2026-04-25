Container Commands
  
    docker run -d nginx → run container in background  
    docker ps → list running containers  
    docker stop <id> → stop container  
    docker rm <id> → remove container  
    docker exec -it <id> bash → enter container  
    docker logs <id> → view logs  


Image Commands

    docker pull nginx → download image  
    docker build -t app:v1 . → build image  
    docker images → list images  
    docker rmi <id> → remove image  
    docker tag app:v1 user/app:v1 → tag image  
    docker push user/app:v1 → push to Docker Hub  
    
Volume Commands

    docker volume create myvol → create volume  
    docker volume ls → list volumes  
    docker volume inspect myvol → details  
    docker volume rm myvol → delete volume  


Network Commands

    docker network create mynet → create network  
    docker network ls → list networks  
    docker network inspect mynet → inspect  
    docker run --network mynet → attach container 
    
Compose Commands

    docker compose up -d → start services  
    docker compose down → stop & remove  
    docker compose ps → list services  
    docker compose logs -f → logs  
    docker compose up --build → rebuild  


Cleanup

    docker system df → disk usage  
    docker system prune → clean unused  

Dockerfile

    FROM → base image  
    RUN → install packages  
    COPY → copy files  
    WORKDIR → set directory  
    EXPOSE → define port  
    CMD → default command  
    ENTRYPOINT → fixed command  


Fix Weak Areas

    docker volume create test
    docker run -v test:/data ubuntu

               OR
          
    docker compose with healthcheck again

