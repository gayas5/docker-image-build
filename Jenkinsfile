pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "gayas555/nginx-app"
        IMAGE_TAG      = "${BUILD_NUMBER}"
        LATEST_TAG     = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                    set -e
                    ls -la
                    [ -f Dockerfile ] || { echo "Dockerfile missing"; exit 1; }
                    [ -f index.html ] || { echo "index.html missing"; exit 1; }
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} .
                    docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:${LATEST_TAG}
                '''
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        set -e
                        echo "Logging in to Docker Hub as $DOCKER_USER"

                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin

                        echo "Pushing images..."
                        docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
                        docker push ${DOCKERHUB_REPO}:${LATEST_TAG}
                    '''
                }
            }
        }
    }

    post {
        always {
            docker logout || true
            docker image prune -f || true
        }
        success {
            echo "✅ Image pushed successfully: ${DOCKERHUB_REPO}:${IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed – check Docker login or permissions"
        }
    }
}
