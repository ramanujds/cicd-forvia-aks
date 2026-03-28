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
				--platform linux/amd64,linux/arm64 \
				-f Dockerfile .

				'''
			}
		}

		stage('Push Image') {
			steps {
				withCredentials([usernamePassword(credentialsId: 'e4af9f44-e2b9-4253-b040-14b40090e1a6', passwordVariable: 'docker_password', usernameVariable: 'docker_user')]) {
					sh '''
					export PATH=$PATH:/usr/local/bin:/opt/homebrew/bin
					echo "$docker_password" | docker login -u "$docker_user" --password-stdin
					docker push ${IMAGE_NAME}:${IMAGE_TAG}
					docker push ${IMAGE_NAME}:latest
					docker logout
					'''
				}
			}
		}




	}
}