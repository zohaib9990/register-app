pipeline {
    agent any

    tools {
        jdk 'java17'
        maven 'maven3'
    }

    environment {
        APP_NAME   = "register-app-pipeline"
        RELEASE    = "1.0.0"
        IMAGE_NAME = "zohaib56/${APP_NAME}"
        IMAGE_TAG  = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM') {
            steps {
                git branch: 'master', credentialsId: 'github', url: 'https://github.com/zohaib9990/register-app.git'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test Application') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkinsa-sonarqube-token') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkinsa-sonarqube-token'
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        def docker_image = docker.build("${IMAGE_NAME}")
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    sh """
                        docker run --rm \
                          -v /var/run/docker.sock:/var/run/docker.sock \
                          aquasec/trivy image ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
    }
}
