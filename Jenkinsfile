pipeline {
    agent { label "Jenkins-Agent" }
    tools {
	    jdk 'Java17'
	    maven 'Maven3'
    }
    environment {
	    APP_NAME = "register-app-pipeline"
	    RELEASE = "1.0.0"
	    DOCKER_USER ="thoratsunil121"
	    DOCKER_PASS ="dockerhub"
	    IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
	    IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }
    stages {
         stage("Cleanup Workspace"){
		 steps{
               	 cleanWs()
		 }
	 }
	 stage("Checkout from SCM"){
		 steps {
		     git branch: 'master', credentialsId: 'gitHub', url: 'https://github.com/thoratsunil/register-app.git'
		 }
	 }
	 stage("Build Application"){
	     steps {
		 sh "mvn clean package"
	     }
          }
	 stage("Test Application"){
	      steps {
	            sh "mvn test"
              }
	  }
	 stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'jenkinsa-sonarqube-token') { 
                        sh "mvn sonar:sonar"
		        }
	           }	
              }
          }
	  stage("Quality Gate"){
	      steps {
		  script {
			waitForQualityGate abortPipeline: false, credentialsId: 'jenkinsa-sonarqube-token'
                  }
	      }
	  }
	  stage("Build & Push Docker Image"){
	      steps {
		 script {
		      docker.withRegistry('',DOCKER_PASS) {
		          docker_image = docker.build "${IMAGE_NAME}"
		       }

		      docker.withRegistry('',DOCKER_PASS) {
			  docker_image.push("${IMAGE_TAG}")
			  docker_image.push('latest')
		      }	      
		 }
	      }
	  
	  }
	    stage("Trivy Scan"){
		steps {
		    script {
			sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image thoratsunil121/register-app-pipeline:latest --no-progress --scanners --reset vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
		    }		    
                }
           }
	    stage("Cleanup Artifacts") {
		steps {
		    script {
			sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
			sh "docker rmi ${IMAGE_NAME}:latest"
    		    }
		}
	    }
	    stage("Trigger CD Pipeline") {
		steps {
                    script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-35-154-126-187.ap-south-1.compute.amazonaws.com:8080/job/gitops-register-app-cd/JENKINS_URL/job/gitops-register-app-cd/build?token=TOKEN_NAME'"
                    }  
               }
	    }
      }    
      post {
            failure {
             emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                      subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
                      mimeType: 'text/html',to: "vistaemailservice@gmail.com"
        }
        success {
            emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                     subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
                     mimeType: 'text/html',to: "vistaemailservice@gmail.com"
      }      
   }
}
