pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'king094/banking-and-finance:v1.0.0'
        DOCKER_REMOTE_DIR = '/home/ubuntu/app'
        TERRAFORM_REMOTE_DIR = '/home/ubuntu/terraform'
        ANSIBLE_REMOTE_DIR = '/home/ubuntu/ansible'
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        ANSIBLE_MACHINE_IP = ''
        TERRAFORM_IP = ''
        DOCKER_IP = ''
        
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
                    sh '''
                        mkdir -p temp_app
                        scp -o StrictHostKeyChecking=no target/*.jar Dockerfile ubuntu@{DOCKER_IP}:/home/ubuntu/app/
                        ssh ubuntu@{DOCKER_IP} "cd /home/ubuntu/app && docker build -t ${DOCKER_IMAGE} ."
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sshagent(['my-ssh-key']) {
                    withCredentials([string(credentialsId: 'docker-hub-password', variable: 'DOCKER_PASSWORD')]) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no ubuntu@{DOCKER_IP} bash -c "'
                                echo \\"$DOCKER_PASSWORD\\" | docker login -u king094 --password-stdin &&
                                docker push king094/banking-and-finance:v1.0.0 && docker logout
                            '"
                        '''
                    }
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                sshagent(['my-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@{TERRAFORM_IP} "
                        cd ${TERRAFORM_REMOTE_DIR} &&
                        ls -l
                        export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                        export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                        terraform init &&
                        terraform apply -auto-approve
                        EC2_IP=$(terraform output -raw ec2_public_ip)
                        echo "[all]" > inventory.ini
                        echo "$EC2_IP" >> inventory.ini
                        scp -o StrictHostKeyChecking=no inventory.ini ubuntu@${ANSIBLE_MACHINE_IP}:${ANSIBLE_REMOTE_DIR}/inventory.ini
                        "
                        '''
                        }
                    }
                }
        stage('Deploy via Ansible') {
            steps {
                sshagent(['my-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@{ANSIBLE_MACHINE_IP} '
                    cd ${ANSIBLE_REMOTE_DIR} &&
                    ansible-playbook -i inventory.ini deploy_docker.yml
                    '
                """
                }
            }
        }
    }
}
