pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'diallof2/lab3-app'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
    }

    stages {
        stage('Lint HTML') {
            steps {
                echo 'Installing HTMLHint and linting index.html...'
                sh 'npm install -g htmlhint'
                sh 'htmlhint index.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml"
                sh "kubectl apply -f deployment-dev.yaml"
                sh "kubectl apply -f ingress.yaml"
            }
        }
    }
}
