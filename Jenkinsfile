pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "vaibhav411007/newbank-backend"
        UI_IMAGE = "vaibhav411007/newbank-ui"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clean') {
            steps { cleanWs() }
        }

        stage('Checkout Backend') {
            steps {
                git branch: 'main',
                url: 'https://github.com/iamvaibhavsutar/NewBank.git'
            }
        }

        stage('Build & Push Backend') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    docker build -t $BACKEND_IMAGE:$IMAGE_TAG .
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $BACKEND_IMAGE:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Checkout UI') {
            steps {
                git branch: 'main',
                url: 'https://github.com/iamvaibhavsutar/NewBank-UI.git'
            }
        }

        stage('Build & Push UI') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    docker build -t $UI_IMAGE:$IMAGE_TAG .
                    docker push $UI_IMAGE:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([string(credentialsId: 'k8s-token', variable: 'TOKEN')]) {
                    sh '''
                    kubectl config set-cluster k8s \
                    --server=https://192.168.49.2:8443 \
                    --insecure-skip-tls-verify=true

                    kubectl config set-credentials jenkins --token=$TOKEN

                    kubectl config set-context k8s-context \
                    --cluster=k8s \
                    --user=jenkins

                    kubectl config use-context k8s-context

                    kubectl apply -f k8s/

                    kubectl set image deployment/newbank-backend \
                    backend=$BACKEND_IMAGE:$IMAGE_TAG

                    kubectl set image deployment/newbank-ui \
                    ui=$UI_IMAGE:$IMAGE_TAG
                    '''
                }
            }
        }
    }

    post {
        success { echo "✅ Production Deployment Successful" }
        failure { echo "❌ Pipeline Failed" }
    }
}
