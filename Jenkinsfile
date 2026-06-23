pipeline {
agent any

environment {
    IMAGE_NAME = "mruthul2373/react-ci-cd"
    IMAGE_TAG = "${BUILD_NUMBER}"
}

stages {

    stage('Build Docker Image') {
        steps {
            sh 
            docker build -t $IMAGE_NAME:$IMAGE_TAG .
            docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
            
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
                sh 
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                docker push $IMAGE_NAME:$IMAGE_TAG
                docker push $IMAGE_NAME:latest
                
            }
        }
    }

    stage('Deploy') {
        steps {
            sh 
            docker stop react-app || true
            docker rm react-app || true

            docker pull $IMAGE_NAME:latest

            docker run -d \
            --name react-app \
            -p 81:80 \
            $IMAGE_NAME:latest
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


