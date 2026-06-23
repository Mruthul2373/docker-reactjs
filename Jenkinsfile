pipeline {
    agent any

    environment {
        IMAGE_NAME = "mruthul2373/react-ci-cd"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push mruthul2373/react-ci-cd:${BUILD_NUMBER}
                    docker push mruthul2373/react-ci-cd:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker stop react-app || true
                docker rm react-app || true
                docker pull mruthul2373/react-ci-cd:latest
                docker run -d --name react-app -p 81:80 mruthul2373/react-ci-cd:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful'
        }
        failure {
            echo 'Deployment Failed'
        }
    }
}
