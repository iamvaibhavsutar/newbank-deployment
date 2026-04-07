pipeline {
    agent any

    stages {

        stage('Pull Latest Images') {
            steps {
                sh 'docker pull vaibhav411007/newbank-app:latest'
                sh 'docker pull vaibhav411007/newbank-ui:latest'
            }
        }

        stage('Deploy via Docker Compose') {
            steps {
                sh '''
                cd /home/vaibhav/newbank-docker || exit 1
                docker-compose down
                docker-compose up -d
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Completed!'
        }
        failure {
            echo '❌ Deployment Failed!'
        }
    }
}
