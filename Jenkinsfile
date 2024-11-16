pipeline {

    agent any

    tools { 
        maven 'my-maven' 
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql')
        SONARQUBE_ENV = credentials('sonarqube-token') 
    }
    stages {

        // stage('Run SonarQube Container') {
        //     steps {
        //         script {
        //             // Run SonarQube container if it's not running
        //             sh '''
        //                 if [ ! "$(docker ps -q -f name=sonarqube)" ]; then
        //                     if [ "$(docker ps -aq -f status=exited -f name=sonarqube)" ]; then
        //                         # Cleanup if a stopped container exists
        //                         docker rm sonarqube
        //                     fi
        //                     # Run the SonarQube container
        //                     docker run -d --name sonarqube -p 9000:9000 --network dev sonarqube:latest
        //                 fi
        //             '''
                    
        //             // Lấy token từ API của SonarQube
        //             def tokenResponse = sh(script: '''
        //                 // apk add --no-cache curl
        //                 curl -u admin:admin -X POST "http://localhost:9000/api/user_tokens/generate?name=my_token"
        //             ''', returnStdout: true).trim()


        //             // Parse response để lấy token
        //             def tokenJson = readJSON text: tokenResponse
        //             SONARQUBE_TOKEN = tokenJson.token
        //             echo "SonarQube Token: ${SONARQUBE_TOKEN}"
        //         }
        //     }
        // }

        // stage('SonarQube Analysis') {
        //     steps {
        //         withSonarQubeEnv('sonarqube scanner') { 
        //             sh '''
        //                 mvn sonar:sonar \
        //                 -Dsonar.projectKey=my-springboot-app \
        //                 -Dsonar.host.url=${SONARQUBE_URL} \
        //                 -Dsonar.login=${SONARQUBE_TOKEN} \
        //                 -Dsonar.java.binaries=target/classes
        //             '''
        //         }
        //     }
        // }

        // stage('Quality Gate') {
        //     steps {
        //         script {
        //             def qg = waitForQualityGate()
        //             if (qg.status != 'OK') {
        //                 error "Pipeline failed due to quality gate failure: ${qg.status}"
        //             }
        //         }
        //     }
        // }

        stage('Packaging/Pushing Image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t tiendn274/springboot .'
                    sh 'docker push tiendn274/springboot'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop tiendoan-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm tiendoan-mysql-data || echo "no volume"'

                sh "docker run --name tiendoan-mysql --rm --network dev -v tiendoan-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
                sh 'sleep 20'
                sh "docker exec -i tiendoan-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull tiendn274/springboot'
                sh 'docker container stop tiendoan-springboot || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '
                sh 'docker container run -d --name tiendoan-springboot -p 8081:8080 --network dev tiendn274/springboot'
            }
        }

    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}
