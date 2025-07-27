pipeline {
    agent any

    tools {
        maven 'Maven3'
        terraform 'Terraform'
    }

    environment {
        // This assumes 'aws-credentials' is of type "AWS Credentials"
        AWS_ACCESS_KEY_ID     = credentials('aws-credentials')
        AWS_SECRET_ACCESS_KEY = credentials('aws-credentials')
    }

    stages {
        stage('Checkout the Code from GitHub') {
            steps {
                git url: 'https://github.com/Abhisheksingh67126/Banking-and-Finance', branch: 'master'
                echo 'Checked out repository.'
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
                sh 'docker build -t king094/banking-and-finance:v1.0.0 .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials',
                                                  usernameVariable: 'DOCKER_USERNAME',
                                                  passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push king094/banking-and-finance:v1.0.0'
            }
        }

        stage('Create Test Infrastructure (Terraform)') {
            steps {
                dir('test') {
                    sh 'chmod 600 KEY-PAIR-POC.pem'
                    sh 'terraform init'
                    sh 'terraform validate'
                    sh 'terraform apply --auto-approve'
                }
            }
        }

        stage('Ansible Deployment') {
            steps {
                ansiblePlaybook credentialsId: 'AnsibleCred',
                                disableHostKeyChecking: true,
                                inventory: 'test/Inventory',
                                playbook: 'ansible-playbook.yml'
            }
        }
    }
}
