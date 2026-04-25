pipeline {
    agent any

    environment {
        AWS_REGION     = 'us-east-1'
        AWS_ACCOUNT_ID = '234201875500'
        ECR_REPO       = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/devops-static-site"
        IMAGE_TAG      = "${BUILD_NUMBER}"
        ECS_CLUSTER    = 'devops-cluster'
        ECS_SERVICE    = 'devops-task-service-7is00v4b'
    }

    stages {
        stage('Checkout Code') {
            steps { checkout scm }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t devops-static-site:${IMAGE_TAG} ."
            }
        }
        stage('Login to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} \\
                    | docker login --username AWS --password-stdin ${ECR_REPO}
                """
            }
        }
        stage('Push to ECR') {
            steps {
                sh "docker tag devops-static-site:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}"
                sh "docker tag devops-static-site:${IMAGE_TAG} ${ECR_REPO}:latest"
                sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
                sh "docker push ${ECR_REPO}:latest"
            }
        }
        stage('Deploy to ECS') {
            steps {
                sh """
                    aws ecs update-service \\
                        --cluster ${ECS_CLUSTER} \\
                        --service ${ECS_SERVICE} \\
                        --force-new-deployment \\
                        --region ${AWS_REGION}
                """
            }
        }
    }
    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed. Check logs above.' }
    }
}
