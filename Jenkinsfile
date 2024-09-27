pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "agon13"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/Codeodata/register-app'
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
		        withSonarQubeEnv(credentialsId: 'jenkins-sonar-token') { 
                        sh "mvn sonar:sonar"
		        }
	           }	
           }
       }
       stage("Build & Push Docker Image"){
           steps {
	        script {
            // Build the Docker image
         		def docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
			
			docker.withRegistry('', 'dockerhub') {

			
            
            // Tag the image correctly with the full name
            		sh "docker push ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}"

            // Optional Tag latest
           		// sh "docker tag ${DOCKER_USER}/${APP_NAME}:latest"

	           }
           }
       }
    } 
  }
}

