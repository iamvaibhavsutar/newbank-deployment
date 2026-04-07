pipeline {
    agent any

    stages {

        stage('Clean Workspace') {
            steps { cleanWs() }
        }

        stage('Checkout Deployment Repo') {
            steps {
                git branch: 'main',
                url: 'https://github.com/iamvaibhavsutar/newbank-deployment.git'
            }
        }

        stage('Deploy via Docker Compose') {
            steps {
                sh '''
                docker-compose down || true
                docker-compose up -d
                '''
            }
        }
    }

    post {
        success { echo "✅ Deployment Success" }
        failure { echo "❌ Deployment Failed" }
    }
}
