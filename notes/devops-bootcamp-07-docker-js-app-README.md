[Notes overview](../README.md)

# 7 - Containers with Docker

## 8 - Developing with Docker

This demo app shows a simple user profile app set up using 
- index.html with pure js and css styles
- nodejs backend with express module
- mongodb for data storage

All components are docker-based

### Before starting to work with a Docker project setup everything

#### Step 1: Install docker through apt

    sudo apt update

download the GNU Privacy Guard file for docker and register it

    sudo apt install ca-certificates curl gnupg
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg

add the docker package repository to ubuntu

    echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update

installing docker    

    sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

testing if installation worked

    sudo docker run hello-world

error: hello-world image was not existing locally and downloading it was not possible
solution: adding the company proxy to docker following the guide https://docs.docker.com/network/proxy/
docker is now usable and working as expected

#### Step 2: Learning about the features of docker

is a combination of docker pull <image-name> (if it doesn't exist locally) and docker start <image-name>

    docker run <image-name>

without any additional parameters start a new container in the current terminal blocking it

stopping it through CTRL+C does not delete the container itself, rerunning the command results in another container creation
reusing the container is possible through using its container id OR by passing the parameter --name=<container-name> and afterwards running through docker start <container-name> or docker start <container-id>
using the parameter -d for detach results in not blocking the terminal and keeping the container running in the back

    docker run <image-name>

list all currently running containers using the parameter -a also lists terminated but existing containers, this can be used to find container id or container name

    docker ps

terminating the container

    docker stop <container-id>
    docker stop <container-name>

can be used to forward the port 6001 of the host system to the port 6379 of the docker container
while multiple docker containers can use the same port, the host cannot, therefore running multiple containers of a image is possible as long as the host provides different ports

    docker run -p 6001:6379 <image-name> 
        
can be used to connect to the bash of a container

    docker exec -it <image-name> /bin/bash

### Starting with the demo project

pulls the mongo database image from docker hub

    docker pull mongo
        
pulls the mongo-express graphical web ui for a mongo database from docker hub

    docker pull mongo-express
        
lists all the existing networks, there are a few by default which will be explained later

    docker network ls
        
will create another bridge network

    docker network create <network-name>
        
basic command to run the mongo db, pass some environment variables (the user information), ports, network and name

    docker run -p 27017:27017 -d -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password --name mongodb --net mongo-network mongo
        
same command but with intendention to make it more readable 

    docker run -d \
    -p 27017:27017 \
    -e MONGO_INITDB_ROOT_USERNAME=admin \
    -e MONGO_INITDB_ROOT_PASSWORD=password \
    --name mongodb \
    --net mongo-network \
    mongo

 to see the logs of a container

    docker logs <container-id>

connection works and through localhost:8081 mongo-express is accessable, but the basic auth information is required to access (see docker logs of the container)

    docker run -d \
    -p 8081:8081 \
    -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
    -e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
    --name mongo-express \
    --net mongo-network \
    -e ME_CONFIG_MONGODB_SERVER=mongodb \
    mongo-express
        
access mongo express through http://localhost:8081
manually create a database "user-account" over the UI (this could also be done through a ENV variable (-e ME_CONFIG_MONGODB_AUTH_DATABASE=user-account)

    cd app
    npm install 
    node server.js

nodejs application is now accessable through http://localhost:3000
changing data through the form results in updates within mongodb, can be checked through the UI but also CLI
    
 to get the container id

    docker ps

 to see the logs of the container, additionally with the parameter -f, so that the logs will be streamed (also meaning the terminal is locked for that task)

    docker logs <container-id>

## 9 - Docker Compose - Run multiple Docker containers

### Step 1: start mongodb and mongo-express

check if any containers are still running

    docker ps
    docker stop <container-id> or <container-name> to stop all containers

starts two containers for mongodb and mongoexpress but also initializes a new network
naming convention depending on the parent folder (in this case app, therefore app_default network)

    cd app
    docker-compose -f mongo.yaml up

error: docker-compose was not installed
solution:

    sudo apt update
    sudo apt install docker-compose
    
_You can access the mongo-express under localhost:8080 from your browser_
    
### Step 2: in mongo-express UI - create a new database "user-account"

### Step 3: in mongo-express UI - create a new collection "users" in the database "user-account"       
    
### Step 4: start node server 

    cd app
    npm install
    node server.js
    
### Step 5: access the nodejs application from browser 

http://localhost:3000

### Step 6: shutdown the application

Stop the node execution through ctrl+c in the terminal

Shutdown the docker containers for mongodb and mongo express
Even removes the created containers and the default network which was created through "up"

    docker-compose -f mongo.yaml down

##  10 - Dockerfile - Build your own Docker Image 

creating a new image based on the Dockerfile called my-app in version 1.0
for it to work properly the node app must be installed within the Dockerfile

    docker build -t my-app:1.0 .
        
Dockerfile was changed therefore the image must be rebuilt, we have to remove the image and the container before we can rebuild the image

find the docker container for my-app

    docker ps -a | grep my-app
        
find the docker image for my-app

    docker images

cannot be executed before the container was removed
    
    docker rm <container-id>
    docker rmi <image-id>

run the built image without any options

    docker run <image-name>:<image-tag>
    
check for the container id so that we can connect on it

    docker ps
        
connecting to the container and calling the bash, if it doesn't exist try /bin/sh

    docker exec -it <container-id> /bin/bash

## 11 - Private Docker Repository

within our nexus installation we create a new repository "docker-hosted" http://207.154.243.149:8081/repository/docker-hosted/
additionally we create a new Role under Security which is able to handle this new repository
we assign the Role to our previously created User

for docker repository we have to add another free port in the http field of the repository were docker can directly connect to

to check wether the port was opened we can switch to our nexus servers shell

    root@ubuntu-s-4vcpu-8gb-fra1-01:~# netstat -lnpt
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 127.0.0.1:44601         0.0.0.0:*               LISTEN      3362/java           
    tcp        0      0 0.0.0.0:8083            0.0.0.0:*               LISTEN      3362/java           
    tcp        0      0 0.0.0.0:8081            0.0.0.0:*               LISTEN      3362/java           
    tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      340026/systemd-reso 
    tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      340026/systemd-reso 
    tcp6       0      0 :::22                   :::*                    LISTEN      1/systemd  

we still need to allow the port 8083 on our firewall configuration of digitalocean droplet
under our droplet => networking => Firewalls "my-droplet-firewall" we can achiev that

last thing to configure is a realm under Security => Realms => Docker Bearer Token Realm

    cat ~/.docker/config.json

if the file doesn't exist this means we've never logged in to a docker repository (docker hub, ...)
after the first login the file should exist

our nexus configuration is currently only running in http
therefore we must allow docker to connect to http as normaly https is forced

on windows/mac this file is not easily accessable because of docker desktop is fully virtualized
on linux we add or edit daemon.json

    vim /etc/docker/daemon.json
    #{
    #   "insecure-registries" : [ "207.154.243.149:8083" ]
    #}

reload docker

    systemctl reload docker

login (with username + password), will be stored unencrypted

    docker login 207.154.243.149:8083

go to the directory with Dockerfile in it

    docker build -t my-app:1.0 .

basically renames the image

    docker tag my-app:1.0 207.154.243.149:8083:/my-app:1.0

pushes the image layer by layer to the repository
if the version of the image is omitted latest is default, the push doesn't work if there is no latest

    docker push 207.154.243.149:8083/my-app:1.0

obvious learning about ssh
the ssh private key does only match to the ssh public key if i'm logged in as the correct user
if my public key is based on "basti" but i'm logged in as root access is obviously denied

we want to setup nexus as a docker container
also we want the data to be persistant and not get lost so we create a volume

    docker volume create --name nexus-data
    docker volume ls

run a container with the nexus3 image, expose port 8081 and use the created volume

    docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3

check the created volume on the host
lists information like the path to the volume on the host
can also be used for containers

    docker inspect nexus-data
    cd /var/snap/docker/common/var-lib-docker/volumes/nexus-data/_data

the data can also be accessed from within the container after connecting to it through docker exec
    
rebuilding changes we made to server.js

    docker build -t my-app:1.0 .

starting the application including mongodb, mongo-express and my-app
still have to manually create a database and collection in mongo-express

added named volume mongo-data which results in a directory /var/lib/docker/volume/<volumnname>
the volume can now be used within other containers through adding them with this name like mongo-data:/data/db in mongodb container

    docker-compose -f mongo.yaml up

## 17 - Docker Best Practises

1. always use official images for example for node, do not install linux + node yourself
2. always use a specific version tag, do not use latest as it is unpredictable
3. if full blown os like ubuntu is not required as all the extra tools are not relevant for the current case, just use a leaner linux distribution for example alpine
4. the order of steps within the dockerfile matter, for a node application we do not want to execute npm install when not necessary, this can be achieved with the correct order check the history of an image, all image layers basically
    ```
    docker history my-app:1.0
    ```
5. you can use .dockerignore to specifically exclude some files from the docker build
6. multi-stage-build use temporary images and only keep the final image
7. always use a least privileged user, never use root, you can just add a group and user, give the correct permissions and change the user within the container to the created one
8. scan vulnerabilities of the image through "docker scan cves", this gives hints on actions you have to do and stores the information for all other users to docker hub
