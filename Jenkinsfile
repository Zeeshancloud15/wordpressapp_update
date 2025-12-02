pipeline {
    agent any

    environment {
        AWS_CREDS = credentials('aws-credentials')          // AWS Access Key + Secret Key in Jenkins
        REGION    = 'us-east-1'
        ECR_REG   = '189598237274.dkr.ecr.us-east-1.amazonaws.com'
        REPO_NAME = 'zeeshan.agency'
        IMAGE_TAG = "jenkins-${env.BUILD_NUMBER}"          // Unique tag per build
        KUBE_NAMESPACE = 'default'                         // Change if you use a custom namespace
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
                    export AWS_ACCESS_KEY_ID=$AWS_CREDS_USR
                    export AWS_SECRET_ACCESS_KEY=$AWS_CREDS_PSW
                    export AWS_DEFAULT_REGION=$REGION

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

                        # Update the deployment image using your Kubernetes YAML
                        kubectl set image deployment/wordpressapp-deployment wordpressapp=$ECR_REG/$REPO_NAME:$IMAGE_TAG -n $KUBE_NAMESPACE --record
                        kubectl rollout status deployment/wordpressapp-deployment -n $KUBE_NAMESPACE
                    """
