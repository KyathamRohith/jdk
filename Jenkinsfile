pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "rohith1305"
        SERVICES = ["shopfront", "productcatalogue", "stockmanager"]
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
                    for (service in SERVICES) {
                        sh """
                        echo "Building and testing ${service}..."
                        mvn -f ${service}/pom.xml clean package
                        """
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    for (service in SERVICES) {
                        sh """
                        echo "Building Docker image for ${service}..."
                        docker build -t $DOCKER_REGISTRY/${service}:latest -f ${service}/Dockerfile ${service}
                        """
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                        for (service in SERVICES) {
                            sh """
                            echo "Pushing ${service} image to Docker Hub..."
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
                    for (service in SERVICES) {
                        sh """
                        echo "Deploying ${service} to Kubernetes..."
                        kubectl apply -f kubernetes/${service}-deployment.yaml
                        kubectl apply -f kubernetes/${service}-service.yaml
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
