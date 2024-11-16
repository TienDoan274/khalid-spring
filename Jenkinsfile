pipeline {
    agent any

    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql')
        SONARQUBE_URL = "http://localhost:9000"  // SonarQube URL trỏ tới container đang chạy
        SONARQUBE_TOKEN = ""  // Chưa có token ở đây, sẽ lấy trong script
    }

    stages {
        stage('Run SonarQube Container') {
            steps {
                script {
                    // Run SonarQube container if it's not running
                    sh '''
                        if [ ! "$(docker ps -q -f name=sonarqube)" ]; then
                            if [ "$(docker ps -aq -f status=exited -f name=sonarqube)" ]; then
                                # Cleanup if a stopped container exists
                                docker rm sonarqube
                            fi
                            # Run the SonarQube container
                            docker run -d --name sonarqube -p 9000:9000 --network dev sonarqube:latest
                        fi
                    '''
                    
                    // Lấy token từ API của SonarQube
                    def tokenResponse = sh(script: '''
                        curl -u admin:admin_password -X POST "http://localhost:9000/api/user_tokens/generate?name=my_token"
                    ''', returnStdout: true).trim()

                    // Parse response để lấy token
                    def tokenJson = readJSON text: tokenResponse
                    SONARQUBE_TOKEN = tokenJson.token
                    echo "SonarQube Token: ${SONARQUBE_TOKEN}"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube scanner') { 
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=my-springboot-app \
                        -Dsonar.host.url=${SONARQUBE_URL} \
                        -Dsonar.login=${SONARQUBE_TOKEN} \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline failed due to quality gate failure: ${qg.status}"
                    }
                }
            }
        }

        // Các stage khác (Packaging, Deploy, v.v...)
    }

    post {
        always {
            cleanWs()
        }
    }
}
