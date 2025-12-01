pipeline {
agent any
stages {
stage('Git Checkout') {
steps {
git branch: 'main', url: 'https://github.com/your/repo.git'
}
}
stage('Build Docker Image') {
steps {
sh 'docker build -t wordpress-app .'
}
}
stage('Login ECR') {
steps {
sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS
--password-stdin <account>.dkr.ecr.ap-south-1.amazonaws.com'
}
}
stage('Push Image') {
steps {
sh 'docker tag wordpress-app:latest <ECR-URI>:blue'
sh 'docker push <ECR-URI>:blue'
}
}
stage('Deploy via Ansible') {
steps {
ansiblePlaybook playbook: 'ansible-deploy.yaml'
}
}
}
}
