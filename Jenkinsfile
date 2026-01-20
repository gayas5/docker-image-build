pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "gayas555/nginx-app"  // Update to your actual Docker Hub repo name
        IMAGE_TAG      = "${env.BUILD_NUMBER}" // e.g., #5, #6
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
                    ls -la
                    test -f Dockerfile   || { echo "ERROR: Dockerfile missing"; exit 1; }
                    test -f index.html   || { echo "ERROR: index.html missing – add it to repo root!"; exit 1; }
                '''
            }
        }

        stage('Build Image') {
            steps {
                script {
                    // Build the image using shell (no plugin needed)
                    sh """
                        docker build -t \${DOCKERHUB_REPO}:\${IMAGE_TAG} .
                    """
                    // Tag as latest
                    sh """
                        docker tag \${DOCKERHUB_REPO}:\${IMAGE_TAG} \${DOCKERHUB_REPO}:\${LATEST_TAG}
                    """
                }
            }
        }

        stage('Push Images') {
            when { expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' } }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',  // Ensure this credential exists in Jenkins
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
            echo "Successfully pushed ${DOCKERHUB_REPO}:${IMAGE_TAG} and :${LATEST_TAG}"
        }
        failure {
            echo "Build or push failed"
        }
    }
}
