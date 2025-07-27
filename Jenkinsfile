pipeline {
    agent any

    stages {
        stage('Checkout the Code from GitHub') {
            steps {
                git url: 'https://github.com/Abhisheksingh67126/Banking-and-Finance'
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
                echo 'Building Docker image: king094/banking-and-finance:v1.0.0'
                sh 'docker build -t king094/banking-and-finance:v1.0.0 .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    echo 'Logging into Docker Hub...'
                    sh '''
                        echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                    '''
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo 'Pushing Docker image: king094/banking-and-finance:v1.0.0'
                sh 'docker push king094/banking-and-finance:v1.0.0'
            }
        }

        stage('Run Test Pipeline from YAML') {
            steps {
                script {
                    def testPipeline = readYaml file: 'Jenkinsfile.yml'
                    echo "Loaded test pipeline config: ${testPipeline}"
                }
            }
        }

        stage('Create Infrastructure') {
            steps {
                dir('test') {
                    echo 'Creating infrastructure with Terraform...'
                    sh 'chmod 600 keypair-.pem'
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
