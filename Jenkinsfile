pipeline {
    agent any

    environment {
        AWS_CREDS = credentials('aws-credentials')
        REGION    = 'us-east-1'
        ECR_REG   = '189598237274.dkr.ecr.us-east-1.amazonaws.com'
        REPO_NAME = 'zeeshan.agency'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git url: 'https://github.com/Zeeshancloud15/wordpressapp_update.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                  docker build -t $REPO_NAME:$IMAGE_TAG .
                """
            }
        }

        stage('Login to ECR & Tag Image') {
            steps {
                sh """
                    aws configure set aws_access_key_id $AWS_CREDS_USR
                    aws configure set aws_secret_access_key $AWS_CREDS_PSW
                    aws configure set default.region $REGION

                    aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ECR_REG
                    docker tag $REPO_NAME:$IMAGE_TAG $ECR_REG/$REPO_NAME:$IMAGE_TAG
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
                  docker push $ECR_REG/$REPO_NAME:$IMAGE_TAG
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_PATH')]) {
                    sh """
                        export KUBECONFIG=$KUBECONFIG_PATH
                        kubectl set image deployment/wordpressapp-deployment \
                          wordpressapp=$ECR_REG/$REPO_NAME:$IMAGE_TAG --record
                        kubectl rollout status deployment/wordpressapp-deployment
                    """
                }
            }
        }
    }
}
