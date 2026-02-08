pipeline {
    agent any

    environment {
        IMAGE_NAME = "youssefaelmansy/java_app"
        TAG = "${BUILD_NUMBER}"
        GIT_REPO = "https://github.com/youssef-elmansy/argocd-repo-task25.git"
    }

    stages {

        stage('Build App') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DockerHub',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                      echo $PASS | docker login -u $USER --password-stdin
                      docker push ''' + IMAGE_NAME + ':' + TAG
                }
            }
        }

        stage('Remove Local Image') {
            steps {
                sh "docker rmi ${IMAGE_NAME}:${TAG}"
            }
        }

        stage('Update deployment.yaml') {
            steps {
                sh '''
                git clone ${GIT_REPO}
                cd gitops-repo
                sed -i "s|image:.*|image: ${IMAGE_NAME}:${TAG}|g" deployment.yaml
                git add deployment.yaml
                git commit -m "Update image to ${TAG}"
                git push
                '''
            }
        }
    }
}

