## STEPS TO RUN THIS APPLICATION

step 1:

write a docker compose file:

docker-compose.yml:

      version: "3.8"
      
      services:
      
          app:
            build: .
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
              test: ["CMD" , "mongosh", "--eval", "db.adminCommand('ping')"]
              interval: 5s
              retries: 5
      
      volumes:
        mongo_data:
      
      networks:
        app-net:


step 2:

.env:

      MONGO_URI=mongodb://mongo:27017/mydb

step 3:

   execute:

     docker compose up -d --build

   check docker image name:

     docker images

   check the port the application running on:

     docker ps 

   go to browser and give the public Ip or localhost Ip and the port number:

     http://localhost:3000 --> you application

   
