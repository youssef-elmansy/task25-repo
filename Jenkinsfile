pipeline {
    agent any

    environment {
        IMAGE_NAME = "youssefaelmansy/java_app"
        TAG = "${BUILD_NUMBER}"
        GITOPS_REPO = "https://github.com/youssef-elmansy/argocd-repo-task25.git"
    }

    stages {

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build App') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'DockerHub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh """
                      echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                      docker push ${IMAGE_NAME}:${TAG}
                    """
                }
            }
        }

        stage('Delete Local Image') {
            steps {
                sh "docker rmi ${IMAGE_NAME}:${TAG} || true"
            }
        }

        stage('Update deployment.yaml (GitOps)') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'GitHub',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_PASS'
                    )
                ]) {
                    sh """
                    rm -rf argocd-repo-task25
                    git clone https://\$GIT_USER:\$GIT_PASS@github.com/youssef-elmansy/argocd-repo-task25.git
                    cd argocd-repo-task25
                    sed -i 's|image:.*|image: ${IMAGE_NAME}:${TAG}|' deployment.yaml
                    git add deployment.yaml
                    git commit -m "Update image to ${TAG}"
                    git push
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished!"
        }
        success {
            echo "Image pushed & GitOps repo updated successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs!"
        }
    }
}

