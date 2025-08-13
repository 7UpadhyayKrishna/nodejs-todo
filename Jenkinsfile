pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '897722705527.dkr.ecr.ap-south-1.amazonaws.com/to-do'
        IMAGE_TAG = "nodeapp-${env.BUILD_NUMBER}"
        PRIVATE_IP = "10.0.2.128"
        PEM_KEY = "/home/ec2-user/test1.pem" // Jenkins server par key ka exact path
    }

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/7UpadhyayKrishna/nodejs-todo.git'
            }
        }

        stage('Remove Old Container & Image') {
            steps {
                sh '''
                docker rm -f nodeapp || true
                docker rmi $(docker images -q ${ECR_REPO}) || true
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${ECR_REPO}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh """
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login --username AWS --password-stdin 897722705527.dkr.ecr.ap-south-1.amazonaws.com
                    """
                }
            }
        }


        stage('Push to ECR') {
            steps {
                sh '''
                docker push ${ECR_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy on Private EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no -i ${PEM_KEY} ec2-user@${PRIVATE_IP} "
                    aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login --username AWS --password-stdin 897722705527.dkr.ecr.ap-south-1.amazonaws.com &&
                    docker rm -f nodeapp || true &&
                    docker pull ${ECR_REPO}:${IMAGE_TAG} &&
                    docker run -d --name nodeapp -p 3000:3000 ${ECR_REPO}:${IMAGE_TAG}
                "
                '''
            }
        }
    }
}
