pipeline {
    agent any

    environment {
        // Java and Maven paths
        JAVA_HOME = "/usr/lib/jvm/java-11-openjdk-amd64"
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
                    // Extract version from pom.xml without readMavenPom
                    def TAG = sh(returnStdout: true, script: "grep -m1 '<version>' pom.xml | sed -e 's/.*<version>\\(.*\\)<\\/version>.*/\\1/'").trim()
                    echo "Building Docker image with tag: ${TAG}"

                    // Check if any JAR exists in target using shell command
                    def jarExists = sh(script: 'ls target/*.jar', returnStatus: true) == 0
                    if (!jarExists) {
                        error("Spring Boot JAR not found in target. Build failed or artifact missing.")
                    }

                    sh "docker build -t petclinic:${TAG} ."
                }
            }
        }

        stage("Start Docker Container") {
            steps {
                script {
                    def TAG = sh(returnStdout: true, script: "grep -m1 '<version>' pom.xml | sed -e 's/.*<version>\\(.*\\)<\\/version>.*/\\1/'").trim()
                    echo "Starting Docker container on port 9090"
                    sh """
                        docker rm -f petclinic || true
                        docker run -d --name petclinic -p 9090:9090 petclinic:${TAG}
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
