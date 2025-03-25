pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "rohith1305"
        KUBE_NAMESPACE = "djshopping"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/KyathamRohith/jdk.git'
            }
        }

        stage('Build and Test Services') {
            steps {
                script {
                    def services = ["shopfront", "productcatalogue", "stockmanager"]
                    for (service in services) {
                        sh """
                        set -e
                        echo "🔨 Building and testing ${service}..."
                        mvn -f ${service}/pom.xml clean package -DskipTests
                        """
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    def services = ["shopfront", "productcatalogue", "stockmanager"]
                    for (service in services) {
                        sh """
                        set -e
                        echo "🐳 Building Docker image for ${service}..."
                        docker build --no-cache -t $DOCKER_REGISTRY/${service}:latest -f ${service}/Dockerfile ${service}
                        """
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    def services = ["shopfront", "productcatalogue", "stockmanager"]
                    withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                        for (service in services) {
                            sh """
                            set -e
                            echo "🚀 Pushing ${service} image to Docker Hub..."
                            docker push $DOCKER_REGISTRY/${service}:latest
                            """
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def services = ["shopfront", "productcatalogue", "stockmanager"]
                    for (service in services) {
                        sh """
                        set -e
                        echo "📦 Deploying ${service} to Kubernetes..."
                        kubectl -n $KUBE_NAMESPACE apply -f kubernetes/${service}-deployment.yaml
                        kubectl -n $KUBE_NAMESPACE apply -f kubernetes/${service}-service.yaml
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ All services deployed successfully!"
        }
        failure {
            echo "❌ Deployment failed. Check logs for errors."
        }
    }
}
