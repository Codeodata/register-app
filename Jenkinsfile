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
	// Log in to Docker Hub using environment variables
           		 sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
            
            // Build the Docker image
         		def docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
            
            // Tag the image correctly with the full name
           		 sh "docker tag ${IMAGE_TAG} ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}"

            // Push the image to Docker Hub
           		 sh "docker push ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}"
           		 sh "docker push ${DOCKER_USER}/${APP_NAME}:latest"

		        docker.withRegistry('',DOCKER_PASS) { 
                        	docker_image = docker.build ("${IMAGE_TAG}")
		        }
	           }
	        script {
		        docker.withRegistry('',DOCKER_PASS) { 
                        	docker_image.push("${IMAGE_TAG}")
		        }
	           }
           }
       }
    } 
}

