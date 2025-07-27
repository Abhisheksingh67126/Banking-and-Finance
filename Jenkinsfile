pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'king094/banking-and-finance:v1.0.0'
        DOCKER_REMOTE_DIR = '/home/ubuntu/app'
        TERRAFORM_REMOTE_DIR = '/home/ubuntu/terraform'
        ANSIBLE_REMOTE_DIR = '/home/ubuntu/ansible'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Abhisheksingh67126/Banking-and-Finance', branch: 'master'
            }
        }

        stage('Code Compile & Test') {
            parallel {
                stage('Compile') {
                    steps {
                        sh 'mvn compile'
                    }
                }
                stage('Test') {
                    steps {
                        sh 'mvn test'
                    }
                }
            }
        }

        stage('Quality Analysis (QA)') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Package Application') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sshagent(['my-ssh-key']) {
                    // Copy project to remote Docker builder
                    sh "scp -o StrictHostKeyChecking=no -r ./ ubuntu@13.126.118.224:${DOCKER_REMOTE_DIR}"

                    // SSH in and build Docker image (with correct variable expansion)
                    sh "ssh ubuntu@13.126.118.224 \"cd ${DOCKER_REMOTE_DIR} && docker build -t ${DOCKER_IMAGE} .\""
                }
            }
        }
    }
}
