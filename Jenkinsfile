pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "793873033616"
        ECR_REPO = "cloudship-app"
        IMAGE_TAG = "latest"
        ECR_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPO}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/akashsubramanian03/cloudship-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                echo 'Building Docker image...'
                docker build -t ${ECR_REPO}:${IMAGE_TAG} .
                """
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([aws(credentialsId: 'aws-creds', region: "${AWS_DEFAULT_REGION}")]) {
                    sh """
                    aws ecr get-login-password --region ${AWS_DEFAULT_REGION} \
                    | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com
                    """
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh """
                docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URL}:${IMAGE_TAG}
                """
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                docker push ${ECR_URL}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@35.154.112.60 "
                            echo 'Stopping existing container...'
                            docker stop \$(docker ps -q) || true &&
                            docker rm \$(docker ps -aq) || true &&

                            echo 'Logging into ECR...'
                            aws ecr get-login-password --region ap-south-1 \
                            | docker login --username AWS --password-stdin 793873033616.dkr.ecr.ap-south-1.amazonaws.com &&

                            echo 'Pulling latest image...'
                            docker pull 793873033616.dkr.ecr.ap-south-1.amazonaws.com/cloudship-app:latest &&

                            echo 'Starting container on port 9090...'
                            docker run -d -p 9090:8080 793873033616.dkr.ecr.ap-south-1.amazonaws.com/cloudship-app:latest
                        "
                    '''
                }
            }
        }
    }
}
