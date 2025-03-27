pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '354918387085'
        AWS_REGION = 'us-east-1'
        ECR_REPO = 'my-app'
        IMAGE_NAME = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
        EC2_USER = "ubuntu"
        EC2_HOST = "devops-key95"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/yosh-avd/weblight-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-app .'
            }
        }

        stage('Push to AWS ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $IMAGE_NAME
                docker tag my-app:latest $IMAGE_NAME:latest
                docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST <<EOF
                docker pull $IMAGE_NAME:latest
                docker stop my-app || true
                docker rm my-app || true
                docker run -d -p 3000:3000 --name my-app $IMAGE_NAME:latest
                EOF
                '''
            }
        }
    }
}
