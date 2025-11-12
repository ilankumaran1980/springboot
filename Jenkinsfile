#!/bin/env groovy

pipeline {
    agent any
    tools {
        maven "maven"   // Jenkins Maven tool name
        jdk "jdk11"     // Jenkins JDK 11 tool name
    }

    stages {

        stage("Maven Build & Test") {
            steps {
                script {
                    sh "mvn -B clean install"
                    junit "**/target/surefire-reports/*.xml"
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    TAG = pom.version
                    sh "docker build -t petclinic:${TAG} ."
                }
            }
        }

        stage("Deploy Docker Container") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    TAG = pom.version
                    sh """
                        # Stop and remove previous container if exists
                        docker rm -f petclinic || true
                        # Run new container on port 9090
                        docker run -d --name petclinic -p 9090:8080 petclinic:${TAG}
                    """
                }
            }
        }
    }
}
