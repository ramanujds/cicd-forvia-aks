pipeline {
	agent any

	environment{
		IMAGE_NAME = "ram1uj/part-inventory-service"
		IMAGE_TAG = "${BUILD_NUMBER}"
	}

	stages {
		stage('Source') {
			steps {
				echo 'Checking out source code from GitHub'
				git branch: 'main', url: 'https://github.com/ramanujds/cicd-forvia-aks'

			}
		}
		stage('Build Docker Image') {
			steps {
				echo 'Building Docker image'
				sh '''
				export PATH=$PATH:/usr/local/bin:/opt/homebrew/bin
				docker build \
				-t ${IMAGE_NAME}:${IMAGE_TAG} \
				-t ${IMAGE_NAME}:latest \
				--platform linux/amd64 .
				'''
			}
		}
	}
}