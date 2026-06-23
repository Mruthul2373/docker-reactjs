pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/Mruthul2373/React-CI-CD.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t mruthul2373/react-ci-cd:${BUILD_NUMBER} .'
                sh 'docker tag mruthul2373/react-ci-cd:${BUILD_NUMBER} mruthul2373/react-ci-cd:latest'
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
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
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

                docker run -d \
                --name react-app \
                -p 80:80 \
                mruthul2373/react-ci-cd:latest
                '''
            }
        }
    }
}

