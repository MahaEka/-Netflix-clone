pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'aisalkyn85'
        IMAGE_NAME = 'spring-boot-app'
        VERSION = 'v1'
    }

    stages {

        stage('Build Spring Boot JAR with Maven') {
            steps {
                echo "⚙️ Building Spring Boot app..."
                dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                    sh 'ls -l'
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                    sh 'docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${VERSION} .'
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo "📤 Pushing image to Docker Hub..."
                sh 'echo "${DOCKERHUB_TOKEN}" | docker login -u ${DOCKERHUB_USER} --password-stdin'
                sh 'docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${VERSION}'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying to Minikube..."
                dir('java-maven-sonar-argocd-helm-k8s/spring-boot-manifests') {
                    sh 'kubectl apply -f deployment.yml'
                    sh 'kubectl apply -f service.yml'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl get pods'
                sh 'kubectl get svc'
            }
        }
    }

    post {
        failure {
            echo "❌ Pipeline failed. Check Jenkins logs for details."
        }
        success {
            echo "✅ Deployment completed successfully!"
        }
    }
}
