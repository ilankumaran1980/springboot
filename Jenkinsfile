pipeline {
    agent any

    environment {
        // You can explicitly set JAVA_HOME and MAVEN_HOME if needed
        JAVA_HOME = "/usr/lib/jvm/java-11-openjdk-arm64"
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
        MAVEN_HOME = "/usr/share/maven"
        PATH = "${env.MAVEN_HOME}/bin:${env.PATH}"
    }

    stages {

        stage("Maven Build & Test") {
            steps {
                echo "Building project with Maven..."
                sh "mvn -B clean install"
                junit "**/target/surefire-reports/*.xml"
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    TAG = pom.version
                    echo "Building Docker image with tag: petclinic:${TAG}"
                    sh "docker build -t petclinic:${TAG} ."
                }
            }
        }

        stage("Deploy Docker Container") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    TAG = pom.version
                    echo "Deploying Docker container on port 9090..."
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
            echo "Pipeline failed."
        }
    }
}
