pipeline {
	agent any

	environment{
		IMAGE_NAME = "ram1uj/part-inventory-service"
		IMAGE_TAG = "${BUILD_NUMBER}"
		GITOPS_REPO_URL = "https://github.com/ramanujds/gitops-repo-forvia.git"
		GITOPS_BRANCH = "main"
		GITOPS_VALUES_FILE = "environments/prod/values/inventory-values.yaml"
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

		stage('Update GitOps Image Tag') {
			steps {
				withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'password', usernameVariable: 'username')]) {
					sh '''
					set -e
					if [ -z "${IMAGE_TAG}" ]; then
						echo "IMAGE_TAG is empty; aborting GitOps update."
						exit 1
					fi
					rm -rf gitops-repo-forvia
					git clone -b ${GITOPS_BRANCH} ${GITOPS_REPO_URL} gitops-repo-forvia
					cd gitops-repo-forvia

					python3 - <<'PY'
					import os
					from pathlib import Path

					tag = os.environ.get("IMAGE_TAG", "").strip()
					values_file = Path(os.environ["GITOPS_VALUES_FILE"])

					if not tag:
						raise SystemExit("IMAGE_TAG is empty; aborting GitOps update.")
					if not values_file.exists():
						raise SystemExit(f"Values file not found: {values_file}")

					lines = values_file.read_text(encoding="utf-8").splitlines()
					in_part = False
					in_image = False
					updated = False

					for i, line in enumerate(lines):
						stripped = line.strip()
						if line.startswith("partInventory:"):
							in_part = True
							in_image = False
							continue

						if in_part and stripped and not line.startswith("  ") and not stripped.startswith("#"):
							in_part = False
							in_image = False
							continue

						if in_part and line.startswith("  image:"):
							in_image = True
							continue

						if in_part and in_image and line.startswith("    tag:"):
							indent = line[: len(line) - len(line.lstrip(" "))]
							lines[i] = f'{indent}tag: "{tag}"'
							updated = True
							break

					if not updated:
						raise SystemExit("Could not find partInventory.image.tag to update.")

					values_file.write_text("\n".join(lines) + "\n", encoding="utf-8")
					PY

					git config user.name "jenkins-bot"
					git config user.email "jenkins-bot@users.noreply.github.com"
					git add "$GITOPS_VALUES_FILE"

					if git diff --cached --quiet; then
						echo "No GitOps changes detected; skipping commit."
					else
						git commit -m "ci: update inventory image tag to ${IMAGE_TAG}"
						git push https://${username}:${password}@github.com/ramanujds/gitops-repo-forvia.git ${GITOPS_BRANCH}
					fi
					'''
				}
			}
		}




	}
}