pipeline {
    agent any

    tools {
        maven 'maven3'     // Use correct name as per Global Tool Configuration
        jdk 'java17'       // Use correct name as per Global Tool Configuration
    }

    environment {
        SONARQUBE_ENV = 'SonarQube' // This must match the name configured in Jenkins > Configure System > SonarQube servers
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/zohaib9990/register-app.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying...'
                // Add deploy steps here
            }
        }
    }
}
