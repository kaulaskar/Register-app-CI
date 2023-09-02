pipeline {
    agent { label 'Jenkins-Agent' }
	
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "register-app"
        RELEASE = "1.0.0"
        DOCKER_USER = "ashfaque9x"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	JENKINS-API-TOKEN = "${JENKINS-API-TOKEN}"
    }
    
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }
        }
		
        stage("Checkout from SCM"){
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Ashfaque-9x/register-app'
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
		    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn sonar:sonar"
		    }
		}	
            }

        }
		
	stage("Quality Gate"){
            steps {
		script {
		    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
		}	
            }
        }

	stage("Build & Push Docker Image") {
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

       stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS-API-TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://43.205.120.95:8080/job/gitops-register-app/buildWithParameters?token=gitops-token'"
                }
            }
       }
   }
}