#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        IMAGE_NAME = "cicd-demo"
        IMAGE_TAG = "latest"
        SONAR_TOKEN = credentials('SONAR_AUTH')
    }

    tools {
        maven 'M3' 
    }

    stages {
        // Get source code
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // Build app
        stage('Build') {
            steps {
                sh "mvn clean compile"
            }
        }

        // Run basic tests
        stage('Test') {
            steps {
                sh "mvn test -DforkCount=0 -Dtest=!SeleniumExampleTest"
            }
        }

        // Source code review
        stage('Static Analysis (SonarQube)') {
            steps {
                script {
                    sh "mvn sonar:sonar \
                        -Dsonar.token=${SONAR_TOKEN} \
                        -Dsonar.projectKey=cicd-demo \
                        -Dsonar.host.url=http://host.docker.internal:9000"
                }
            }
        }

        // Build docker image
        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        // Docker image review
        stage('Container Secutiry Scan (Trivy)') {
            steps {
                sh "docker run --rm \
                    -v /var/run/docker.sock:/var/run/docker.sock \
                    aquasec/trivy image \
                    --exit-code 1 \
                    --severity CRITICAL \
                    ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        // Deployment
        stage('Deploy') {
            when { branch 'master' }

            steps {
                sh "docker rm -f ${IMAGE_NAME} || true"
                sh "docker run -d --name ${IMAGE_NAME} -p 80:80 ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
    }

    post {
        always {
            echo 'Cleaning environment...'

            cleanWs()
        }

        failure {
            echo "Pipeline has stopped"
        }
    }
}
