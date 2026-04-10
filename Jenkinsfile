pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "abhishekf5/cicd-e2e:latest"
        KUBECONFIG = credentials('kubeconfig')
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Nagendra27014/cicd-end-to-end'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image..."
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'USERNAME',
                        passwordVariable: 'PASSWORD'
                    )]) {
                        sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    kubectl apply -f k8s/
                    kubectl set image deployment/myapp myapp=${DOCKER_IMAGE}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS 🚀"
        }

        failure {
            echo "Pipeline FAILED ❌ Check logs"
        }
    }
}
