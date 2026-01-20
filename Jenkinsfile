pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "gayas555/nginx-app"
        IMAGE_TAG      = "${env.BUILD_NUMBER}"
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
                    test -f index.html   || { echo "ERROR: index.html missing"; exit 1; }
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh """
                    docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} .
                    docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:${LATEST_TAG}
                """
            }
        }

        stage('Login & Push') {
            when { expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' } }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh '''
                        # Debug (careful - username is not secret)
                        echo "Logging in as user: $DH_USER"
                        
                        # The secure way
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        
                        # If login succeeds → push
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
            echo "Build or push failed - check logs above (especially docker login step)"
        }
    }
}
