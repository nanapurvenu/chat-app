pipeline {
    agent any

    stages {
        stage('code-analysis') {
            steps {
                echo 'Sonar Analysis Started'
                sh '''
                cd chat-app && sudo docker run --rm \
                -e SONAR_HOST_URL="http://35.88.131.52:9000" \
                -v "$(pwd):/usr/src" \
                -e SONAR_TOKEN="sqp_05bffcd89426e4961e01b5f7206a525ea7066245" \
                sonarsource/sonar-scanner-cli -Dsonar.projectKey=chat-app
                '''
                echo 'Sonar Analysis Completed'
            }
        }

        stage('build containers') {
            steps {
                script {
                    def packageJson = readJSON file: 'chat-app/backend/package.json'
                    def packageJSONVersion = packageJson.version
                    echo "App Version: ${packageJSONVersion}"
                    sh """
                    cd chat-app/backend && \
                    docker build -t venureddy3417/chatapp-be:${packageJSONVersion} . && \
                    docker container run -dt --name chatapp-backend -p 8081:8080 venureddy3417/chatapp-be:${packageJSONVersion}
                    """

                    sh """
                    cd chat-app/frontend && \
                    docker build -t venureddy3417/chatapp-fe:${packageJSONVersion} . && \
                    docker container run -dt --name chatapp-frontend -p 80:80 venureddy3417/chatapp-fe:${packageJSONVersion}
                    """
                }
            }
        }

        stage('docker login & push') {
            steps {
                script {
                    echo 'Logging into Docker Hub and pushing images'
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pwd', usernameVariable: 'user')]) {
                        sh '''
                        sudo docker login -u $user -p $pwd
                        sudo docker push venureddy3417/chatapp-be:${packageJSONVersion}
                        sudo docker push venureddy3417/chatapp-fe:${packageJSONVersion}
                        '''
                    }
                }
            }
        }
    }
}
