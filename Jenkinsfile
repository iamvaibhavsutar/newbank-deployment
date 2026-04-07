pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "vaibhav411007/newbank-backend"
        UI_IMAGE = "vaibhav411007/newbank-ui"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        // ================= CLEAN =================
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        // ================= BACKEND =================
        stage('Checkout Backend') {
            steps {
                dir('backend') {
                    git branch: 'main',
                    url: 'https://github.com/iamvaibhavsutar/NewBank.git'
                }
            }
        }

        stage('Build & Push Backend Image') {
            steps {
                dir('backend') {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )]) {
                        sh '''
                        echo "🔨 Building Backend Image..."
                        docker build -t $BACKEND_IMAGE:$IMAGE_TAG .

                        echo "🔐 Logging into DockerHub..."
                        echo $PASS | docker login -u $USER --password-stdin

                        echo "📤 Pushing Backend Image..."
                        docker push $BACKEND_IMAGE:$IMAGE_TAG
                        '''
                    }
                }
            }
        }

        // ================= UI =================
        stage('Checkout UI') {
            steps {
                dir('ui') {
                    git branch: 'main',
                    url: 'https://github.com/iamvaibhavsutar/NewBank-UI.git'
                }
            }
        }

        stage('Build & Push UI Image') {
            steps {
                dir('ui') {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )]) {
                        sh '''
                        echo "🔨 Building UI Image..."
                        docker build -t $UI_IMAGE:$IMAGE_TAG .

                        echo "📤 Pushing UI Image..."
                        docker push $UI_IMAGE:$IMAGE_TAG
                        '''
                    }
                }
            }
        }

        // ================= DEPLOYMENT =================
        stage('Checkout Deployment Repo') {
            steps {
                dir('deployment') {
                    git branch: 'main',
                    url: 'https://github.com/iamvaibhavsutar/newbank-deployment.git'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir('deployment') {
                    withCredentials([string(credentialsId: 'k8s-token', variable: 'TOKEN')]) {
                        sh '''
                        echo "🔑 Checking Token..."
                        echo "Token length: ${#TOKEN}"

                        echo "⚙️ Setting Kubernetes Config..."
                        kubectl config set-cluster k8s \
                        --server=https://192.168.49.2:8443 \
                        --insecure-skip-tls-verify=true

                        kubectl config set-credentials jenkins --token=$TOKEN

                        kubectl config set-context k8s-context \
                        --cluster=k8s \
                        --user=jenkins

                        kubectl config use-context k8s-context

                        echo "📡 Testing Cluster Access..."
                        kubectl get nodes

                        echo "📂 Checking k8s folder..."
                        ls -l k8s/

                        echo "🚀 Applying Kubernetes Manifests..."
                        kubectl apply -f k8s/ --validate=false

                        echo "🔄 Updating Backend Image..."
                        kubectl set image deployment/newbank-backend \
                        backend=$BACKEND_IMAGE:$IMAGE_TAG

                        echo "🔄 Updating UI Image..."
                        kubectl set image deployment/newbank-ui \
                        ui=$UI_IMAGE:$IMAGE_TAG

                        echo "📊 Checking Deployment Status..."
                        kubectl get pods
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Production Deployment Successful 🚀"
        }
        failure {
            echo "❌ Pipeline Failed - Check Logs 🔍"
        }
    }
}
