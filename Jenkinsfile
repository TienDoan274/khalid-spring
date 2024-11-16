pipeline {

    agent any

    tools { 
        maven 'my-maven' 
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql')
        SONAR_TOKEN = credentials('sonarqube-token') 
    }
    stages {

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }
        stage 'Code Quality'
            sh "mvn sonar:sonar -Dsonar.host.url=http://sonarqube:9000"
        // stage('SonarQube Analysis') {
        //     steps {
        //         withSonarQubeEnv('sonarqube scanner') { // Tên SonarQube Server cấu hình trong Jenkins
        //             sh '''
        //                 mvn sonar:sonar \
        //                 -Dsonar.projectKey=my-springboot-app \
        //                 -Dsonar.host.url=http://sonarqube:9000 \
        //                 -Dsonar.login=${SONAR_TOKEN} \
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
