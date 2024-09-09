[Notes overview](https://github.com/TheAbys/devops-bootcamp-certification-project/blob/master/README.md)

# jenkins introduction

## 2 - Install Jenkins

added a new droplet with a new firewall which allows connection through :8080
connected to the server and install docker

    apt update
    apt install docker.io

installing application through docker
port 50000 is not used in the current case, but is necessary for a cluster setup, multiple worker nodes communicate through that port

    docker run -p 8080:8080 -p 50000:50000 -d \
        -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts

application is now reachable through http://138.68.111.144:8080/

for installation we require the admin password which is stored within the docker container in path /var/jenkins_home/secrets/initialAdminPassword

    docker ps
    docker exec -it <container-hash> bash
    cat /var/jenkins_home/secrets/initialAdminPassword

password can obviously be found on the host if you access the right docker volume folder

1. we install suggested plugins
2. added a new admin user basti
3. started with jenkins

## 3 - Introduktion to Jenkins UI

You basically can divide it into to parts. An administrator part and a DevOps part.
Through Manage Jenkins you can administer the jenkins application.
Through New Item you can build pipelines for all your applications.

## 4 - Install Build Tools in Jenkins

There are two ways.
Installing a build tool through Manage Jenkins => Tools (Maven for example).
Or installing it from the command line on the machine (the container, not the host) itself

    docker exec -u 0 -it <container-hash> bash

Get the current version of os (this helps to find the correct installation guide for the build tools we require)

    cat /etc/issue
    apt update


The script is not up-to-date anymore
    curl -sL https://deb.nodesource.com/setup_20.x -o nodesource_setup.sh

Correct solution (https://github.com/nodesource/distributions)
    apt update
    apt install -y ca-certificates curl gnupg
    mkdir -p /etc/apt/keyrings
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

    NODE_MAJOR=20 // change version to whatever version must be installed
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list
    apt update
    apt install nodejs -y


## 5 - Jenkins Basics Demo - Freestyle Job

We create a new freestyle job and don't change anything
Under Build Steps we add a new Build Step "Execute Shell"

    npm --version

Here we can execute npm because we've installed in directly on the container
Maven for example we did not install that way, therefore we cannot use it here

But we can add a Maven Build Step through Execute Maven Goal where we choose our previously defined Maven and add

    --version

Within Jenkins we can now run Build Now which results in a new build we can open afterwards
Through Console Output we can see what Jenkins executed and with that the version of npm an Maven

If we add the Plugin for NodeJs we get the module available within Build Steps like we did with Maven

Update the Job by adding my git repository.
I've had to use my private key and some extra configuration within Jenkins to connect to my repository.

On Jenkins /var/jenkins_home/jobs i can find all the generated jobs
Under /var/jenkins_home/workspace there is the checked out code from my repository

We've added a new Test AppTest.java and configured the Freestyle Build Job so that it executes

    maven test
    maven package

Within the build console output we see that the tests were executed

    [INFO] -------------------------------------------------------
    [INFO]  T E S T S
    [INFO] -------------------------------------------------------
    [INFO] Running AppTest
    [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.079 s -- in AppTest
    [INFO] 
    [INFO] Results:
    [INFO] 
    [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

## 6 - Docker in Jenkins

We want to use Docker within our Build Jobs.
Therefore we need to make Docker available within Jenkins.

We stop our current container

    docker ps
    docker stop <container-hash>

We can see that the volume we previously created still exists

    docker volume ls

Now we start a new container like before, but we mount the Docker installation into it

    docker run -p 8080:8080 -p 50000:50000 -d \
    -v jenkins_home:/var/jenkins_home \
    -v /var/run/docker.sock:/var/run/docker.sock \
    jenkins/jenkins:lts

Again connect to the container (as root -u 0)

    docker ps
    docker exec -u 0 -it <container-hash> bash

We need to install docker

    curl https://get.docker.com/ > dockerinstall && chmod 777 dockerinstall && ./dockerinstall

And set the right permission

    ls -l /var/run/docker.sock
    chmod 666 /var/run/docker.sock
    exit

login as jenkins

    docker exec -it <container-hash> bash

lets build a Docker image and push it to docker hub or any other repository

    docker build -t <docker-hub-repo>:jma-1.0 .

Login with username and password

    docker login -u $USERNAME -p $PASSWORD

Improved way

    c

Push the image

    docker push <docker-hub-repo>:jma-1.0

### Push Dockerimage to our Nexus Repository

Nexus repository is still unsecure, we have to add insecure-repositories to Docker

    vim /etc/docker/daemon.json

    {
        "insecure-repositories": ["64.226.110.153:8083"]
    }

We now must restart docker

    systemctl restart docker

Our running container was killed when we restarted docker, we have to restart it

    docker ps -a
    docker start <container-hash>

Because we previously made the permission changes in the container and it was restartet we lost the permissions

    docker exec -u 0 -it <container-hash> bash
    ls -l /var/run/docker.sock
    chmod 666 /var/run/docker.sock
    exit

I've realized that I'm connecting to our first Nexus installation and not to the second one with Docker.
I saw that as an opportunity to check my learnings.

I've created a new Docker repository on Nexus and set the http port to 8083
The droplet used the same firewall configuration as the other one therefore i did not have to add the inbound port 8083 here

Afterwards the port 8083 must be exposed so I've stopped the running Nexus container, deleted it and restarted the container with an additional parameter

    docker ps
    docker container rm <container-hash>
    docker run -d -p 8081:8081 -p 8083:8083 --name nexus -v nexus-data:/nexus-data sonatype/nexus3

# 7 - Freestyle to Pipeline Job / 8 - Intro to Pipeline Job

Basically through a scripted pipeline we are versioning the whole build, test and deploy process within our application git repository and we have much more possibilties to handle this process

While it is possible to configure the same or close to the same through freestyle it has very high maintainance cost as everything has to be configured through UI, Plugins, etc.

Doing it through a Jenkinsfile we can write declarative pipelines (see Jenkinsfile in current project) or write it directly with groovy language.
Groovy gives more options but for learning is also more complex than the declarative style.
Groovy also allows the execution of Groovy libraries. Those must be allowed by the Jenkins admin.

# 9 - Jenkinsfile Syntax

See comments within Jenkinsfile
All available environmental variables within Jenkins can be found on http://138.68.111.144:8080/env-vars.html/ after installation

# 10 - Create complete Pipeline

Moved Jenkinsfile to ExampleJenkinsfile
Created new Jenkinsfile

# 13 - Credentials in Jenkins

Credentials can have different types like "Username with Password" or "Secret" or ... and through plugins it is possible to add further types.
The scope defines in which scope the credentials can be used. System => Jenkins server, Global => everywhere, Multi-Branch-Pipeline => project (folder)
Good practise to use credentials on folders.
