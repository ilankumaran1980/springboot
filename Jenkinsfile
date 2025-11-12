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
                    // Make sure the JAR exists
                    def jarExists = fileExists "target/*.jar"
                    if (!jarExists) {
                        error("Spring Boot JAR not found in target. Build failed or artifact missing.")
                    }
                    echo "Building Docker image with tag: ${TAG}"
                    sh "docker build -t petclinic:${TAG} ."
                }
            }
        }

        stage("Start Docker Container") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    TAG = pom.version
                    echo "Starting Docker container on port 9090"
                    sh """
                        docker rm -f petclinic || true
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
