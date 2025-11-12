pipeline {
    agent any

    environment {
        // Java and Maven paths
        JAVA_HOME = "/usr/lib/jvm/java-11-openjdk-arm64"
        MAVEN_HOME = "/usr/share/maven"
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"
    }

    stages {
        stage("Checkout Code") {
            steps {
                echo "Checking out code from GitHub"
                git branch: 'main', url: 'https://github.com/ilankumaran1980/springboot.git'
            }
        }

        stage("Maven Build & Test") {
            steps {
                echo "Building the project with Maven..."
                sh "mvn -B clean install"
                junit "**/target/surefire-reports/*.xml"
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    TAG = pom.version
                    echo "Building Docker image with tag: ${TAG}"
                    sh "docker build -t petclinic:${TAG} ."
                }
            }
        }

        stage("Deploy Docker Container") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    TAG = pom.version
                    echo "Deploying Docker container on port 9090"
                    sh """
                        # Stop & remove old container if exists
                        docker rm -f petclinic || true
                        # Run new container mapping port 9090 to 9090 (make sure Dockerfile exposes 9090)
                        docker run -d --name petclinic -p 9090:9090 petclinic:${TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! Access the app at http://<host-ip>:9090"
        }
        failure {
            echo "Pipeline failed. Check the logs for errors."
        }
    }
}
