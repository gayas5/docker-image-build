pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "gayas555/nginx-app"
        IMAGE_TAG      = "${env.BUILD_NUMBER}"          // e.g. 42
        LATEST_TAG     = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm  // Simpler than explicit git; uses the repo configured in the job
            }
        }

        stage('Build Image') {
            steps {
                script {
                    // Build once, tag multiple times
                    def image = docker.build("${DOCKERHUB_REPO}:${IMAGE_TAG}")
                    image.tag("${LATEST_TAG}")
                }
            }
        }

        stage('Push Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
                        docker push ${DOCKERHUB_REPO}:${LATEST_TAG}
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f || true'
            sh 'docker logout || true'
        }
        success {
            echo "Pushed: ${DOCKERHUB_REPO}:${IMAGE_TAG} and :latest"
        }
    }
}
