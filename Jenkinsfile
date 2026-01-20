pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "gayas555/nginx-app"
        IMAGE_TAG      = "${env.BUILD_NUMBER}"   // e.g. #5, #6, ...
        LATEST_TAG     = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm   // Uses the repo/branch already configured in the job
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
        sh 'docker build -t gayas5/your-app-name:${BUILD_NUMBER} .'
    }
}

stage('Push Images') {
    when { expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' } }
    steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
            sh '''
                echo $PASS | docker login -u $USER --password-stdin
                docker push gayas5/your-app-name:${BUILD_NUMBER}
                docker tag gayas5/your-app-name:${BUILD_NUMBER} gayas5/your-app-name:latest
                docker push gayas5/your-app-name:latest
            '''
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
