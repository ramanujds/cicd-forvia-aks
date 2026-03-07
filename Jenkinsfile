pipeline {
	agent any

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
				-t ram1uj/part-inventory-service:${BUILD_NUMBER} \
				-t ram1uj/part-inventory-service:latest \
				-- platform linux/amd64 \
				-f Dockerfile .
				'''
			}
		}
	}
}