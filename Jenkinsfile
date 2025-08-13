pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_CREDENTIALS = credentials('aws-creds')
        ECR_REPO = '897722705527.dkr.ecr.ap-south-1.amazonaws.com/my-project'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t $ECR_REPO:$IMAGE_TAG .
                """
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds-id']]) {
                    sh """
                    aws configure set aws_access_key_id AKIA5CBDRMZ3TBWWB57T
                    aws configure set aws_secret_access_key 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
                    aws ecr get-login-password --region ap-south-1 \
                    | docker login --username AWS --password-stdin 897722705527.dkr.ecr.ap-south-1.amazonaws.com
                    """
                }

            }
        }


        stage('Push to ECR') {
            steps {
                sh """
                    docker push ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy on Private EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@<PRIVATE_EC2_IP> '
                            aws ecr get-login-password --region ${AWS_REGION} \
                            | docker login --username AWS --password-stdin ${ECR_REPO} &&
                            docker rm -f nodeapp || true &&
                            docker pull ${ECR_REPO}:${IMAGE_TAG} &&
                            docker run -d --name nodeapp -p 3000:3000 ${ECR_REPO}:${IMAGE_TAG}
                        '
                    """
                }
            }
        }
    }
}
