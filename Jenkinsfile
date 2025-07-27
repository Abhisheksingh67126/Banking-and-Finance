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
                    sh '''
                        rm -rf temp_app
                        mkdir temp_app
                        rsync -av --exclude='.git' ./ temp_app/
                        scp -o StrictHostKeyChecking=no -r temp_app ubuntu@13.126.118.224:/home/ubuntu/app
                        ssh ubuntu@13.126.118.224 "cd /home/ubuntu/app && docker build -t ${DOCKER_IMAGE} ."
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sshagent(['my-ssh-key']) {
                    withCredentials([string(credentialsId: 'docker-hub-password', variable: 'DOCKER_PASSWORD')]) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no ubuntu@13.126.118.224 bash -c "'
                                echo \\"$DOCKER_PASSWORD\\" | docker login -u king094 --password-stdin &&
                                docker push king094/banking-and-finance:v1.0.0 &&
                                docker logout
                            '"
                        '''
                    }
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                sshagent(['my-ssh-key']) {
                    withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', 
                     credentialsId: 'aws-credentials', 
                     accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                     secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']
                ]) {
                    sh '''
                        echo "Copying Terraform files to remote machine..."
                        scp -o StrictHostKeyChecking=no -r ./test ubuntu@43.205.191.131:$TERRAFORM_REMOTE_DIR

                        echo "Running Terraform remotely..."
                        ssh -o StrictHostKeyChecking=no ubuntu@43.205.191.131 << EOF
                        export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                        export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                        cd $TERRAFORM_REMOTE_DIR
                        terraform init
                        terraform apply -auto-approve
                    EOF
                '''
            }
        }
    }
}


}
}
