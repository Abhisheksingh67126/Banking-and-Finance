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
        
        stage('Remote Docker Build') {
            steps {
                sshagent(['docker-ssh']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@65.2.188.138 '
                    cd ~/myapp &&
                    docker build -t bankingapp:latest .
                    '
                    '''
                }
            }
        }
        
        stage('Expose Port via Docker') {
            steps {
                echo 'Running Docker container and exposing port...'
                sh 'docker run -dt -p 8091:8091 --name c000 myimg'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }

        always {
            echo 'Cleaning up resources if needed.'
        }
    }
}
