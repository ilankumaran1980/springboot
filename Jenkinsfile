pipeline {
    agent any

    environment {
        // Java and Maven paths
        JAVA_HOME = "/usr/lib/jvm/java-11-openjdk-arm64"
        MAVEN_HOME = "/usr/share/maven"
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"
    }

    stages {
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
                        # Remove old container if exists
                        docker rm -f petclinic || true
                        # Run new container mapping port 9090 on host to 8080 in container
                        docker run -d --name petclinic -p 9090:8080 petclinic:${TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs for errors."
        }
    }
}
