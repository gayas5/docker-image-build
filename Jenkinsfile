pipeline {
    agent any

    environment {
        IMAGE_NAME = "gayas555/nginx-app"
        IMAGE_TAG  = "v1"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/gayas5/docker-image-build.git'
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                    pwd
                    ls -la
                    if [ ! -f index.html ]; then
                        echo "ERROR: index.html is missing in repo root!"
                        exit 1
                    fi
                    if [ ! -f Dockerfile ]; then
                        echo "ERROR: Dockerfile missing!"
                        exit 1
                    fi
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',  // Make sure this credential exists in Jenkins
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Docker Image Built & Pushed Successfully → ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "Docker Image Build or Push Failed"
        }
        always {
            sh 'docker logout || true'  // Clean up login
        }
    }
}
