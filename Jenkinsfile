pipeline {
    agent any

    stages {
        stage('Lint HTML') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                echo 'Installing HTMLHint and linting index.html...'
                sh 'npm install -g htmlhint'
                sh 'htmlhint index.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("diallof2/lab3-app:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.image("diallof2/lab3-app:${BUILD_NUMBER}").push()
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                sh "sed -i 's|diallof2/lab3-app:latest|diallof2/lab3-app:${BUILD_NUMBER}|' deployment-dev.yaml"
                sh "kubectl apply -f deployment-dev.yaml"
                sh "kubectl apply -f ingress.yaml"
            }
        }
    }
}
