pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '283904064946.dkr.ecr.us-east-1.amazonaws.com/june12-dev-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
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

        stage('Docker Push') {
            steps {
                sh '''
                docker tag june12-app:${IMAGE_TAG} $ECR_REPO:${IMAGE_TAG}
                docker push $ECR_REPO:${IMAGE_TAG}
                '''
            }
        }

        stage('Update Helm Image Tag') {
            steps {
                sh '''
                sed -i "s/tag: .*/tag: ${IMAGE_TAG}/" june12-chart/values.yaml
                git config user.email "jenkins@example.com"
                git config user.name "jenkins"
                git add june12-chart/values.yaml
                git commit -m "Update image tag to ${IMAGE_TAG}" || true
                git push origin main
                '''
            }
        }
    }
}
