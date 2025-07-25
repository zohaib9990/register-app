pipeline {
    agent any

    environment {
        // Credential ID for SonarQube token stored in Jenkins
        SONARQUBE_CRED_ID = 'jenkins-sonarqube-token'
    }

    tools {
        // Define Maven and JDK installations (as configured in Jenkins global tools)
        maven 'Maven 3.8.6'         // Replace with your Maven installation name
        jdk 'JDK 11'                // Replace with your JDK installation name
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(installationName: 'MySonarQubeServer', credentialsId: "${SONARQUBE_CRED_ID}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true, credentialsId: "${SONARQUBE_CRED_ID}"
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build and SonarQube analysis successful.'
        }
        failure {
            echo '❌ Build or analysis failed. Check console output.'
        }
        always {
            echo '📦 Jenkins pipeline finished.'
        }
    }
}
