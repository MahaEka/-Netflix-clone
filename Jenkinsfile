pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials-id'   // 🔹 Jenkins credentials ID for Docker Hub
        DOCKERHUB_USER = 'aisalkyn85'                        // 🔹 Your Docker Hub username
        IMAGE_NAME = 'spring-boot-app'
        APP_PATH = 'java-maven-sonar-argocd-helm-k8s/spring-boot-app'
        KUBE_PATH = 'java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests'
    }

    stages {
        stage('Checkout Repository') {
            steps {
                echo '📦 Checking out repository from GitHub...'
                checkout scm
                sh 'ls -l'
            }
        }

        stage('Build Spring Boot JAR with Maven') {
            steps {
                echo '⚙️ Building Spring Boot app...'
                dir("${APP_PATH}") {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                dir("${APP_PATH}") {
                    sh 'docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} .'
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo '🚀 Pushing Docker image to Docker Hub...'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        sh 'docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '☸️ Deploying to Kubernetes...'
                dir("${KUBE_PATH}") {
                    sh '''
                        kubectl apply -f deployment.yml
                        kubectl apply -f service.yml
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo '🔍 Checking Kubernetes deployment status...'
                sh 'kubectl get pods -o wide'
                sh 'kubectl get svc'
            }
        }
    }

    post {
        success {
            echo '✅ Deployment completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check Jenkins logs for details.'
        }
    }
}
