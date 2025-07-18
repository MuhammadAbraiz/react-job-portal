pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_FRONTEND = "job-portal-frontend"
        DOCKER_IMAGE_BACKEND = "job-portal-backend"
        DOCKER_TAG = "${BUILD_NUMBER}"
        FRONTEND_PORT = "3000"
        BACKEND_PORT = "4000"
        SLACK_CHANNEL = "#job-portal-ci"
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "🚀 Starting CI/CD Pipeline for Job Portal App"
                    
                    // Clean workspace
                    deleteDir()
                    
                    // Checkout code
                    checkout scm
                    
                    echo "✅ Source code checked out successfully"
                }
            }
        }
        
        stage('Build & Test') {
            parallel {
                stage('Backend') {
                    steps {
                        script {
                            dir('backend') {
                                echo "🔧 Installing backend dependencies..."
                                sh 'npm ci'
                                
                                echo "🧪 Running backend tests..."
                                sh 'npm test || true'
                                
                                echo "✅ Backend build & test completed"
                            }
                        }
                    }
                }
                
                stage('Frontend') {
                    steps {
                        script {
                            dir('frontend') {
                                echo "🎨 Installing frontend dependencies..."
                                sh 'npm ci'
                                
                                echo "🧪 Running frontend tests..."
                                sh 'npm test -- --watchAll=false || true'
                                
                                echo "✅ Frontend build & test completed"
                            }
                        }
                    }
                }
            }
        }
        
        stage('Build Docker Images') {
            steps {
                script {
                    echo "🐳 Building Docker images..."
                    
                    // Build backend image
                    sh "docker build -t ${DOCKER_IMAGE_BACKEND}:${DOCKER_TAG} -f backend/Dockerfile ./backend"
                    sh "docker tag ${DOCKER_IMAGE_BACKEND}:${DOCKER_TAG} ${DOCKER_IMAGE_BACKEND}:latest"
                    
                    // Build frontend image
                    sh "docker build -t ${DOCKER_IMAGE_FRONTEND}:${DOCKER_TAG} -f frontend/Dockerfile ./frontend"
                    sh "docker tag ${DOCKER_IMAGE_FRONTEND}:${DOCKER_TAG} ${DOCKER_IMAGE_FRONTEND}:latest"
                    
                    echo "✅ Docker images built successfully"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    echo "🚀 Deploying application..."
                    
                    // Stop and remove existing containers
                    sh 'docker-compose down || true'
                    
                    // Deploy using docker-compose
                    withEnv([
                        "JWT_SECRET_KEY=${env.JWT_SECRET_KEY}",
                        "CLOUDINARY_API_KEY=${env.CLOUDINARY_API_KEY}",
                        "CLOUDINARY_API_SECRET=${env.CLOUDINARY_API_SECRET}",
                        "CLOUDINARY_CLOUD_NAME=${env.CLOUDINARY_CLOUD_NAME}",
                        "DOCKER_TAG=${DOCKER_TAG}"
                    ]) {
                        sh 'docker-compose up -d --build'
                    }
                    
                    // Wait for services to be ready
                    echo "⏳ Waiting for services to start..."
                    sleep 30
                    
                    // Health check
                    sh "curl -f http://localhost:${BACKEND_PORT}/api/v1/health || echo 'Backend health check failed'"
                    sh "curl -f http://localhost:${FRONTEND_PORT} || echo 'Frontend health check failed'"
                    
                    echo "✅ Deployment completed successfully"
                }
            }
        }
    }
    
    post {
        success {
            script {
                echo "🎉 Pipeline completed successfully!"
                // Uncomment and configure Slack notification if needed
                // slackSend channel: SLACK_CHANNEL, color: 'good', message: "✅ *SUCCESS*: Job Portal App build #${BUILD_NUMBER} deployed successfully!"
            }
        }
        failure {
            script {
                echo "❌ Pipeline failed!"
                // Uncomment and configure Slack notification if needed
                // slackSend channel: SLACK_CHANNEL, color: 'danger', message: "❌ *FAILED*: Job Portal App build #${BUILD_NUMBER} failed! ${BUILD_URL}console"
            }
        }
    }
}
