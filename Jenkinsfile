pipeline {
    agent any

    environment {
        IMAGE_NAME = "mruthul2373/react-ci-cd"
        IMAGE_TAG = "${BUILD_NUMBER}"

        NEXUS_URL = "http://65.0.179.222:8081"
        NEXUS_REPO = "react-artifacts"
    }

    stages {

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    docker logout || true
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'sonarqube-token',
                        variable: 'SONAR_TOKEN'
                    )
                ]) {
                    sh '''
                    npx sonar-scanner \
                    -Dsonar.projectKey=react-ci-cd \
                    -Dsonar.projectName=react-ci-cd \
                    -Dsonar.sources=src \
                    -Dsonar.host.url=http://65.0.179.222:9000 \
                    -Dsonar.token=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Build React App') {
            steps {
                sh '''
                npm install
                npm run build
                '''
            }
        }

        stage('Create Artifact') {
            steps {
                sh '''
                tar -czf react-build-${BUILD_NUMBER}.tar.gz build/
                '''
            }
        }

        stage('Upload To Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-creds',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS'
                    )
                ]) {
                    sh '''
                    curl -v -u $NEXUS_USER:$NEXUS_PASS \
                    --upload-file react-build-${BUILD_NUMBER}.tar.gz \
                    ${NEXUS_URL}/repository/${NEXUS_REPO}/react-build-${BUILD_NUMBER}.tar.gz
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                DOCKER_BUILDKIT=0 docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                docker push ${IMAGE_NAME}:${IMAGE_TAG}
                docker push ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker stop react-app || true
                docker rm react-app || true

                docker pull ${IMAGE_NAME}:latest

                docker run -d \
                  --name react-app \
                  -p 81:80 \
                  ${IMAGE_NAME}:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'SonarQube + Nexus + DockerHub + Deployment Successful'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}
