pipeline {
    agent any

    environment {
        IMAGE_NAME = "dockerhub_username/react-cicd-app"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install & Test') {
            steps {
                sh '''
                npm install
                npm test -- --watchAll=false
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                docker stop react-app || true
                docker rm react-app || true
                docker run -d -p 80:80 --name react-app $IMAGE_NAME:latest
                '''
            }
        }
    }

    post {
        success {
            echo "✅ BUILD SUCCESSFUL"
        }
        failure {
            echo "❌ BUILD FAILED"
        }
    }
}
