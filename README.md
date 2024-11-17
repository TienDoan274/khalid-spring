# Create simple Spring project with Jenkins and Sonarqube

### Set up Jenkins with Docker

Create a bridge network in Docker
`docker network create jenkins`

Run a docker:dind Docker image

```
docker run --name jenkins-docker --rm --detach ^
  --privileged --network jenkins --network-alias docker ^
  --env DOCKER_TLS_CERTDIR=/certs ^
  --volume jenkins-docker-certs:/certs/client ^
  --volume jenkins-data:/var/jenkins_home ^
  --publish 2376:2376 ^
  docker:dind
```

Create a Dockerfile with the following content (Customize the official Jenkins Docker image):

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
dockerfile
```

Build and run a new docker image from this Dockerfile 

`docker build -t myjenkins-blueocean:2.479.1-1 .`

```
docker run --name jenkins-blueocean --restart=on-failure --detach ^
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 ^
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 ^
  --volume jenkins-data:/var/jenkins_home ^
  --volume jenkins-docker-certs:/certs/client:ro ^
  --publish 8080:8080 --publish 50000:50000 myjenkins-blueocean:2.479.1-1
```

### Set up Sonarqube inside container "jenkins-docker"

`docker exec -it jenkins-docker sh`

Create new network

`docker network create dev`

Run sonarqube docker container

 `docker run -d --name sonarqube -p 9000:9000 --network dev sonarqube:latest`
 
Get Sonarqube token
`apk add curl`
`curl -u admin:admin -X POST "http://localhost:9000/api/user_tokens/generate?name=my_token"`

Save this token to config in Jenkins later !

### Config Jenkins with necessary plugins, tools and credentinals

Jenkins server usually available at url: http://localhost:8080 and wait until the Unlock Jenkins page appears.

`docker exec jenkins-blueocean cat /var/jenkins_home/secrets/initialAdminPassword`

Paste this password into the Administrator password field.

After that, choose install suggested plugins

Install a few more necessary plugins for this project: Dashboard->Manage Jenkins->Plugins->Available plugins: SonarQube Scanner, Docker, Maven Intergration.

Set up credentials at: Dashboard->Manage Jenkins->Credentials
- Docker hub account (id:dockerhub)
- Mysql account (id:mysql)
- SonarQube token (The token that we have made before,id:sonarqube-token)

