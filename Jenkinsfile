pipeline {
    agent any

    environment {
        // If DockerHub or private registry is needed in future
        // DOCKER_REGISTRY = "your-private-registry.com"
        IMAGE_NAME = "petclinic"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/ilankumaran1980/springboot.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh "mvn clean install -DskipTests"  // Skip tests for faster iteration
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Stop Existing Container') {
            steps {
                // Stop previous container if running
                sh '''
                if [ $(docker ps -q -f name=petclinic-container) ]; then
                    docker stop petclinic-container
                    docker rm petclinic-container
                fi
                '''
            }
        }

        stage('Run Docker Container') {
            steps {
                sh "docker run -d --name petclinic-container -p 9090:9090 ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
    }

    post {
        success {
            echo "Petclinic app deployed successfully on port 9090!"
        }
        failure {
            echo "Deployment failed."
        }
    }
}
