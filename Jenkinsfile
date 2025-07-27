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
                    sh "scp -r ./ ubuntu@13.126.118.224:${DOCKER_REMOTE_DIR}"
                    sh "ssh ubuntu@13.126.118.224 'cd ${DOCKER_REMOTE_DIR} && docker build -t ${DOCKER_IMAGE} .'"
                }
            }
        }

        stage('Push Docker Image To Docker-Hub') {
            steps {
                sshagent(['my-ssh-key']) {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                        ssh ubuntu@docker-builder.example.com '
                            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin && \
                            docker push ${DOCKER_IMAGE} && \
                            docker logout
                        '
                        """
                    }
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                sshagent(['my-ssh-key']) {
                    sh "scp -r ./test ubuntu@terraform-node.example.com:${TERRAFORM_REMOTE_DIR}"
                    sh "ssh ubuntu@terraform-node.example.com 'cd ${TERRAFORM_REMOTE_DIR} && terraform init && terraform apply -auto-approve'"
                }
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                sshagent(['my-ssh-key']) {
                    sh "scp -r ./ ubuntu@ansible-node.example.com:${ANSIBLE_REMOTE_DIR}"
                    sh "ssh ubuntu@ansible-node.example.com 'cd ${ANSIBLE_REMOTE_DIR} && ansible-playbook -i test/Inventory ansible-playbook.yml'"
                }
            }
        }
    }
}
