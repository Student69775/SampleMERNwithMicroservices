pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '084375553968'
        ECR_REPOS = ['hello-service', 'profile-service', 'mern-frontend']
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', 
                    credentialsId: 'github-credentials', 
                    url: 'https://github.com/Student69775/SampleMERNwithMicroservices.git'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    '''
                }
            }
        }

        stage('Build & Push Docker Images') {
            steps {
                script {
                    for (repo in ECR_REPOS) {
                        sh """
                        echo "🚀 Building and Pushing ${repo}..."
                        docker build -t ${repo}:${IMAGE_TAG} ${repo}/
                        docker tag ${repo}:${IMAGE_TAG} $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/${repo}:${IMAGE_TAG}
                        docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/${repo}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Cleanup Docker Images') {
            steps {
                script {
                    for (repo in ECR_REPOS) {
                        sh """
                        docker rmi ${repo}:${IMAGE_TAG}
                        docker rmi $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/${repo}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ All Docker images built and pushed successfully!"
        }
        failure {
            echo "❌ Build failed. Check logs."
        }
    }
}
