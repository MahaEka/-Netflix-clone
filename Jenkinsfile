
pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'     // Jenkins credentials ID for Docker Hub
        DOCKER_IMAGE = 'aisalkyn85/spring-boot-app'          // Replace with your Docker Hub repo
        DOCKER_TAG = "v${BUILD_NUMBER}"
        KUBE_DEPLOYMENT = 'java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml'
        KUBE_SERVICE = 'java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/service.yml'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo '📦 Checking out code from GitHub...'
                git branch: 'main', url: 'https://github.com/Aisalkyn85/Jenkins-Zero-To-Hero.git'
                sh 'ls -la'
            }
        }

        stage('Build with Maven') {
            steps {
                echo '⚙️ Building Spring Boot app using Maven...'
                sh '''
                    cd java-maven-sonar-argocd-helm-k8s/spring-boot-app
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                script {
                    sh '''
                        cd java-maven-sonar-argocd-helm-k8s/spring-boot-app
                        docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
                    '''
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo '📤 Pushing Docker image to Docker Hub...'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        sh "docker push $DOCKER_IMAGE:$DOCKER_TAG"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '🚀 Deploying to Kubernetes (Minikube)...'
                script {
                    sh '''
                        kubectl apply -f $KUBE_DEPLOYMENT
                        kubectl apply -f $KUBE_SERVICE
                        kubectl rollout status deployment/spring-boot-app
                        kubectl get pods -o wide
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check Jenkins logs for details.'
        }
    }
}
