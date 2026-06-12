pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '283904064946.dkr.ecr.us-east-1.amazonaws.com/june12-dev-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        EKS_CLUSTER = 'june12-dev-eks'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t june12-app:${IMAGE_TAG} .'
            }
        }

        stage('ECR Login') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Docker Tag & Push') {
            steps {
                sh '''
                docker tag june12-app:${IMAGE_TAG} $ECR_REPO:${IMAGE_TAG}
                docker tag june12-app:${IMAGE_TAG} $ECR_REPO:latest
                docker push $ECR_REPO:${IMAGE_TAG}
                docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                    aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER
                    kubectl apply -f k8s/
                    kubectl rollout status deployment/june12-app
                    '''
                }
            }
        }

        stage('Show ALB URL') {
            steps {
                sh '''
                echo "Waiting for ALB DNS..."
                sleep 60
                kubectl get ingress june12-app-ingress
                '''
            }
        }
    }
}
