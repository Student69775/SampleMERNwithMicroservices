pipeline {
    agent any
    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '084375553968'
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
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh '''
                        echo "🔑 Logging into AWS ECR..."
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    '''
                }
            }
        }
        stage('Build & Push Docker Images') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    script {
                        // First, list the directory contents to debug
                        sh "ls -la"
                        sh "ls -la backend || echo 'backend directory not found'"
                        sh "ls -la frontend || echo 'frontend directory not found'"
                        
                        // Define services and their paths
                        def services = [
                            [name: 'hello-service', path: 'backend/hello-service'],
                            [name: 'profile-service', path: 'backend/profile-service'],
                            [name: 'mern-frontend', path: 'frontend']
                        ]
                        
                        // Build and push each service
                        for (service in services) {
                            def serviceName = service.name
                            def servicePath = service.path
                            
                            sh """
                            echo "🔍 Checking path for ${serviceName}..."
                            if [ -d "${servicePath}" ]; then
                                echo "🚀 Building and Pushing ${serviceName}..."
                                docker build -t ${serviceName}:${IMAGE_TAG} ${servicePath}/
                                docker tag ${serviceName}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${serviceName}:${IMAGE_TAG}
                                docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${serviceName}:${IMAGE_TAG}
                            else
                                echo "⚠️ Directory ${servicePath} not found. Skipping ${serviceName}."
                            fi
                            """
                        }
                    }
                }
            }
        }
        stage('Cleanup Docker Images') {
            steps {
                script {
                    def ECR_REPOS = ['hello-service', 'profile-service', 'mern-frontend']
                    for (repo in ECR_REPOS) {
                        sh """
                        echo "🧹 Cleaning up local images for ${repo}..."
                        docker rmi ${repo}:${IMAGE_TAG} || true
                        docker rmi ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${repo}:${IMAGE_TAG} || true
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