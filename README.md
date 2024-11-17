# Simple Spring Project with Jenkins and SonarQube

This project demonstrates how to set up a simple Spring application, integrate it with Jenkins for Continuous Integration (CI), and use SonarQube for static code analysis.

---

## Table of Contents

1. [Set up Jenkins with Docker](#set-up-jenkins-with-docker)
2. [Set up SonarQube with Docker](#set-up-sonarqube-with-docker)
3. [Configure Jenkins with Necessary Plugins and Credentials](#configure-jenkins-with-necessary-plugins-and-credentials)
4. [Create and Run Jenkins Pipeline](#create-and-run-jenkins-pipeline)
5. [Test the Application](#test-the-application)

---

## Set up Jenkins with Docker

### 1. Create a Bridge Network in Docker

First, create a custom network for Jenkins and Docker containers to communicate.

```
docker network create jenkins
```
### 2. Run docker:dind Image
To allow Jenkins to interact with Docker, run the docker:dind (Docker-in-Docker) image:

```
docker run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  --publish 8081:8081 \
  docker:dind
```

### 3.Create Dockerfile for Jenkins
Create a Dockerfile for a customized Jenkins image with Docker support:

```
FROM jenkins/jenkins:2.479.1-jdk17
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
```

### 4. Build and Run the Custom Jenkins Image
Build the Docker image and run the Jenkins container with the following commands:

```
docker build -t myjenkins-blueocean:2.479.1-1 .
docker run --name jenkins-blueocean --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  --publish 8080:8080 --publish 50000:50000 \
  myjenkins-blueocean:2.479.1-1
```
## Set up SonarQube with Docker
### 1. Run SonarQube Container
Run the SonarQube container in the same network as Jenkins:

```
docker run -d --name sonarqube -p 9000:9000 --network jenkins sonarqube:latest
SonarQube will be available at http://localhost:9000.
```
### 2. Generate SonarQube Token
Create a SonarQube token by navigating to My Account -> Security -> Generate Tokens. Save this token for later use in Jenkins.

### 3. Create Webhook in SonarQube
Navigate to Administration -> Configuration -> Webhooks and create a webhook with the following URL: http://jenkins-blueocean:8080/sonarqube-webhook

## Configure Jenkins with Necessary Plugins and Credentials
### 1. Unlock Jenkins
Jenkins will be available at http://localhost:8080. Wait for the "Unlock Jenkins" page to appear.

Get the Administrator password:

```
docker exec jenkins-blueocean cat /var/jenkins_home/secrets/initialAdminPassword
Once logged in, choose Install suggested plugins.
```

### 2. Install Necessary Plugins
Go to Dashboard -> Manage Jenkins -> Plugins -> Available and install the following plugins:
- SonarQube Scanner
- Docker
- Maven Integration
### 3. Set Up Credentials
Navigate to Dashboard -> Manage Jenkins -> Credentials to configure the following credentials:

- Docker hub account (ID: dockerhub)
- MySQL account (ID: mysql)
- SonarQube token (ID: sonarqube-token, use the token generated in SonarQube)
### 4. Set Up Tools
Navigate to Dashboard -> Manage Jenkins -> Tools and configure the following tools:

Maven (Name: my-maven, set default version)
### 5. Configure SonarQube in Jenkins
Go to Dashboard -> Manage Jenkins -> System and configure the SonarQube servers:

SonarQube Server Name: sonarqube scanner

## Create and Run Jenkins Pipeline
### 1. Create a New Multibranch Pipeline
Go to Jenkins Dashboard, click New Item, and choose Multibranch Pipeline.
Add Branch Source and select Git.
Fill in the repository URL of your project on GitHub.
Click Apply and then Save.

### 2. Trigger Build Process
The pipeline will now trigger the build process. Wait until the pipeline finishes running.

### 3. Test the Application
Once the pipeline completes successfully, you can test your Spring application using the following API endpoints:

Show all users:
http://localhost:8081/demo/all

Create a new user:
http://localhost:8081/demo/add?name=tiendoan&email=doannhattien4@gmail.com&age=20
