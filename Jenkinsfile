pipeline {
    agent any

    environment {
        // Disable BuildKit to avoid buildx issues
        DOCKER_BUILDKIT = "0"

        // Docker Hub credentials ID in Jenkins
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'

        // Docker image name
        IMAGE_NAME = 'abhishekf5/cicd-e2e'
        IMAGE_TAG = 'latest'

        // Kubernetes manifests repo
        K8S_REPO = 'https://github.com/Nagendra27014/k8s-manifests.git'
        K8S_BRANCH = 'main'
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                git url: 'https://github.com/Nagendra27014/cicd-end-to-end', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    echo "Building Docker Image..."
                    docker build -t $IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "$DOCKER_CREDENTIALS_ID", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:$IMAGE_TAG
                        docker logout
                        '''
                    }
                }
            }
        }

        stage('Checkout K8S Manifests') {
            steps {
                git url: "$K8S_REPO", branch: "$K8S_BRANCH"
            }
        }

        stage('Update K8S Deployment') {
            steps {
                script {
                    // Replace image tag in deployment.yaml
                    sh '''
                    sed -i "s|image:.*|image: $IMAGE_NAME:$IMAGE_TAG|" deployment.yaml
                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins CI"
                    git add deployment.yaml
                    git commit -m "Update deployment image to $IMAGE_TAG"
                    git push origin $K8S_BRANCH
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
