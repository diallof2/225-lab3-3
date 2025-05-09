pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'                            // DockerHub credentials in Jenkins
        DOCKER_IMAGE = 'diallof2/lab3-app'                                   // Your Docker image
        IMAGE_TAG = "build-${BUILD_NUMBER}"                                  // Unique tag per build
        GITHUB_URL = 'https://github.com/diallof2/225-lab3-3.git'            // Your GitHub repo
        KUBECONFIG = credentials('roseaw-225')                               // Your kubeconfig credential ID
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }

        stage('Lint HTML') {
            steps {
                sh 'npm install htmlhint --save-dev'
                sh 'npx htmlhint *.html'
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
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                script {
                    def kubeConfig = readFile(KUBECONFIG)
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml"
                    sh "kubectl apply -f deployment-dev.yaml"
                    sh "kubectl apply -f ingress.yaml"
                }
            }
        }

        stage('Check Kubernetes Cluster') {
            steps {
                sh "kubectl get pods"
                sh "kubectl get services"
                sh "kubectl get deploy"
            }
        }
    }

    post {
        always {
            node {
                sh '''
                curl -X POST -H 'Content-type: application/json' \
                --data '{"text":"✅ Build Completed: ${JOB_NAME} #${BUILD_NUMBER}"}' \
                https://hooks.slack.com/services/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
                '''
            }
        }
    }
}
