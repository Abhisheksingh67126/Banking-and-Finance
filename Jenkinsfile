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
       Stage('build docker image') {
           steps{
                script{
                    sh 'docker build -t king094/banking-project:v1.0.0 .'
                }
           }
       }
    }
}
