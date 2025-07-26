pipeline {
    agent any
    
    stages {
        stage('Checkout the Code from GitHub') {
            steps {
                git url: 'https://github.com/Abhisheksingh67126/Banking-and-Finance/'
                echo 'Code checked out from GitHub.'
            }
        }
        
        stage('Code Compile') {
            steps {
                echo 'Starting compile...'
                sh 'mvn compile'
            }
        }
        
        stage('Code Testing') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }
        
        stage('Quality Analysis (QA)') {
            steps {
                echo 'Running code style checks...'
                sh 'mvn checkstyle:checkstyle'
            }
        }
        
        stage('Package Application') {
            steps {
                echo 'Packaging application...'
                sh 'mvn package'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'sudo docker build -t king094/banking-project:v1.0.0 .'
                }
            }
        }
        
        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin'
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    sh 'docker push king094/banking-project:v1.0.0'
                }
            }
        }
        stage('Deploy Docker Image via Ansible') {
            steps {
                script {
                    echo 'Deploying Docker image using Ansible playbook...'
                    sh """
                       ansible-playbook -i ansible-playbook.yml
                    """
                }
            }
        }
    }
}
