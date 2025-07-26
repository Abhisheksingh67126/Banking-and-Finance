pipeline{
    agent any
    stages{
        stage('checkout the code from github'){
            steps{
                 git url: 'https://github.com/Abhisheksingh67126/Banking-and-Finance/'
                 echo 'github url checkout'
            }
        }
        stage('codecompile'){
            steps{
                echo 'starting compiling'
                sh 'mvn compile'
            }
        }
        stage('codetesting'){
            steps{
                sh 'mvn test'
            }
        }
        stage('qa'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage('package'){
            steps{
                sh 'mvn package'
            }
        }
        stage('run dockerfile'){
          steps{
              stage('remote docker build')
              steps{
                  sshagent(['docker-ssh-key']){
                      sh '''
                      ssh -o StrictHostKeyChecking=no ubuntu@65.2.188.138 '
                      cd ~/myapp &&
                      docker-build -t bankingapp:latest .
                      '
                      '''
           }
         }
        stage('port expose'){
            steps{
                sh 'docker run -dt -p 8091:8091 --name c000 myimg'
            }
        }   
    }
}
