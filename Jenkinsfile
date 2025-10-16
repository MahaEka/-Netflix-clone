pipeline {
    agent any

    environment {
        APP_NAME      = "netflix-clone"
        IMAGE_NAME    = "netflix-clone"
        IMAGE_TAG     = "v1"
        DOCKERFILE    = "Dockerfile"
        BUILD_CONTEXT = "."
        K8S_PATH      = "Kubernetes"
    }

    stages {
        stage('Checkout Repository') {
            steps {
                echo "📦 Checking out repository..."
                git branch: 'main', url: 'https://github.com/Aisalkyn85/Netflix-clone.git'
                sh 'ls -R'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "📥 Installing dependencies..."
                sh '''
                    if [ -f yarn.lock ]; then
                        yarn install
                    else
                        npm install
                    fi
                '''
            }
        }

        stage('Build React App') {
            steps {
                echo "🏗️ Building Netflix Clone app..."
                sh '''
                    if [ -f yarn.lock ]; then
                        yarn build
                    else
                        npm run build
                    fi
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f ${DOCKERFILE} ${BUILD_CONTEXT}
                '''
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    echo "🧹 Cleaning old containers..."
                    sh '''
                        docker rm -f ${APP_NAME} || true
                        docker run -d -p 8080:80 --name ${APP_NAME} ${IMAGE_NAME}:${IMAGE_TAG}
                        docker ps
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying to Kubernetes..."
                sh '''
                    kubectl apply -f ${K8S_PATH}/deployment.yml
                    kubectl apply -f ${K8S_PATH}/service.yml
                    kubectl get pods -o wide
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Netflix Clone successfully built and deployed!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
