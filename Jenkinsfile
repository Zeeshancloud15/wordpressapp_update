pipeline {
    agent any

    environment {
        REGION    = 'us-east-1'
        ECR_REG   = '189598237274.dkr.ecr.us-east-1.amazonaws.com'
        REPO_NAME = 'zeeshan.agency'
        IMAGE_TAG = "jenkins-${env.BUILD_NUMBER}"  // Unique image tag per build
        KUBE_NAMESPACE = 'default'                // Change if using a custom namespace
    }

    stages {

        stage('Checkout from GitHub') {
            steps {
                git branch: 'main', url: 'https://github.com/Zeeshancloud15/wordpressapp_update.git'
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
                    # Login using EC2 role (no credentials needed)
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
                sh """
                    # Use EC2 instance role automatically
                    export AWS_DEFAULT_REGION=$REGION

                    # Generate kubeconfig dynamically using EC2 role
                    aws eks update-kubeconfig --name floral-jazz-mongoose --region $REGION

                    # Update deployment image and wait for rollout
                    kubectl set image deployment/wordpressapp-deployment wordpressapp=$ECR_REG/$REPO_NAME:$IMAGE_TAG -n $KUBE_NAMESPACE
                    kubectl rollout status deployment/wordpressapp-deployment -n $KUBE_NAMESPACE
                """
            }
        }
    }

    post {
        success {
            echo "✅ Build, push to ECR, and deploy to EKS successful!"
        }
        failure {
            echo "❌ Pipeline failed! Check the logs for errors."
        }
    }
}
