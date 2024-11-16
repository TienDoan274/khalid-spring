pipeline {

    agent any

    tools { 
        maven 'my-maven' 
    }

    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql')
        SONARQUBE_ENV = credentials('sonarqube_token') 
    }

    stages {

        stage('Build with Maven') {
            steps {
                powershell 'mvn --version'
                powershell 'java -version'
                powershell 'mvn clean package -D maven.test.failure.ignore=true'
            }
        }

        // stage('SonarQube Analysis') {
        //     steps {
        //         withSonarQubeEnv('SonarQube') { // Tên SonarQube Server cấu hình trong Jenkins
        //             powershell '''
        //                 mvn sonar:sonar ^
        //                 -Dsonar.projectKey=my-springboot-app ^
        //                 -Dsonar.host.url=http://localhost:9000 ^
        //                 -Dsonar.login=${SONARQUBE_ENV} ^
        //                 -Dsonar.java.binaries=target/classes
        //             '''
        //         }
        //     }
        // }

        // stage('Quality Gate') {
        //     steps {
        //         // Đợi và kiểm tra kết quả Quality Gate từ SonarQube
        //         script {
        //             def qg = waitForQualityGate()
        //             if (qg.status != 'OK') {
        //                 error "Pipeline failed due to quality gate failure: ${qg.status}"
        //             }
        //         }
        //     }
        // }

        stage('Packaging/Pushing image') {

            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    powershell 'docker build -t tiendn274/springboot .'
                    powershell 'docker push tiendn274/springboot'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                powershell 'docker image pull mysql:8.0'
                powershell 'docker network create dev || echo "this network exists"'
                powershell 'docker container stop tiendoan-mysql || echo "this container does not exist" '
                powershell 'echo y | docker container prune '
                powershell 'docker volume rm tiendoan-mysql-data || echo "no volume"'

                powershell "docker run --name tiendoan-mysql --rm --network dev -v tiendoan-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
                powershell 'sleep 20'
                powershell "docker exec -i tiendoan-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                powershell 'docker image pull tiendn274/springboot'
                powershell 'docker container stop tiendoan-springboot || echo "this container does not exist" '
                powershell 'docker network create dev || echo "this network exists"'
                powershell 'echo y | docker container prune '

                powershell 'docker container run -d --name tiendoan-springboot -p 8081:8080 --network dev tiendn274/springboot'
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
