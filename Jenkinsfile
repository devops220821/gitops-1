pipeline {
    agent any
    tools { 
        maven 'maven3.8.8' 
    }
    options {
        buildDiscarder logRotator( 
            daysToKeepStr: '2', 
            numToKeepStr: '2'
        )
    }
	environment {
        DOCKERHUB_USERNAME = "rathaiah"
        APP_NAME = "gitops-demo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub-cred'
	}
    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    cleanWs()
                    sh """
                    echo "Cleaned Up Workspace for ${JOB_NAME}"
                    """
                }
            }
        }
        stage('Checkout SCM'){
            steps {
                git url: 'https://github.com/devops220821/gitops-1.git',
                branch: 'main'
            }
        }
		stage('Build Docker Image'){
            steps {
                script{
                    docker_image = docker.build "${IMAGE_NAME}"
                }
            }
        }
        stage('Push Docker Image'){
            steps {
                script{
                    docker.withRegistry('', REGISTRY_CREDS ){
                        docker_image.push("${BUILD_NUMBER}")
                        //docker_image.push('latest')
                    }
                }
            }
        }
        stage('Delete Docker Images'){
            when {
                expression { true }
            }
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                //sh "docker rmi ${IMAGE_NAME}:latest"
			}	
		}
		stage('Triger confige Changes Pipeline'){
		    steps {
		      sh "curl -v -k --user dev:111926c810e95c254ec636e9b79149e5ca -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://192.168.242.137:8080/job/gitops-py1/buildWithParameters?token=gitops-py1'"   
		    }
		}
    }
}