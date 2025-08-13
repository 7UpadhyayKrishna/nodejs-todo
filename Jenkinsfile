pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '897722705527.dkr.ecr.ap-south-1.amazonaws.com/to-do'
        IMAGE_TAG = "nodeapp-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/7UpadhyayKrishna/nodejs-todo.git/'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $ECR_REPO:$IMAGE_TAG ."
            }
        }

        stage('Remove Old Image & Container') {
            steps {
                sh """
                docker rm -f nodeapp || true
                docker rmi $(docker images -q $ECR_REPO) || true
                """
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region $AWS_REGION \
                | docker login --username AWS --password-stdin 897722705527.dkr.ecr.ap-south-1.amazonaws.com
                """
            }
        }


        stage('Push to ECR') {
            steps {
                sh "docker push $ECR_REPO:$IMAGE_TAG"
            }
        }

        stage('Deploy on Private EC2') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no -i /home/ec2-user/test1.pem ec2-user@10.0.2.128 '
                aws ecr get-login-password --region $AWS_REGION \
                | docker login --username AWS --password-stdin 897722705527.dkr.ecr.ap-south-1.amazonaws.com &&
                docker rm -f nodeapp || true &&
                docker pull $ECR_REPO:$IMAGE_TAG &&
                docker run -d --name nodeapp -p 3000:3000 $ECR_REPO:$IMAGE_TAG
                '
                """
            }
        }

    }
}
