pipeline {
    agent {label "slave1"}
    environment {
    TIME = sh(script: 'date "+%Y-%m-%d %H:%M:%S"', returnStdout: true).trim()
  }
    stages {
        stage('Checkout SCM'){
            steps{
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs:[[
                        url: 'https://github.com/sivanmarom/flask_app.git',
                        credentialsId: ''
                        ]]
                    ])
            }
        }
        
        stage('Build Docker image') {
            steps {
                sh "cp app/Dockerfile ."
                sh "cp app/requirements.txt ."
                sh 'sudo docker build  -t flask_image .'
              sh "sudo docker run -d -p 5000:5000 flask_image"
            }
        }
        stage("build user") {
        steps{
              wrap([$class: 'BuildUser']) {
                sh 'echo ${BUILD_USER} >> Result.json'
              }
        }
        }
       stage ("testing"){
              steps{
                      sh  ' curl -I $(dig +short myip.opendns.com @resolver1.opendns.com):5000 | grep "HTTP/1.1 200 OK" >> Result.json'
   
               sh 'echo "$TIME" >> Result.json'
             sh "aws dynamodb execute-statement --statement \"INSERT INTO test-result VALUE { 'user':'$BUILD_USER', 'date':'$TIME', 'state': '\$(curl -I \$(dig +short myip.opendns.com @resolver1.opendns.com):5000 | grep \"HTTP/1.1 200 OK\")' }\""


    
                         }
         }
      
        
        
        stage ('upload to s3 bucket'){
            steps{
                withAWS(credentials: 'awscredentials'){
                     sh 'aws s3 cp Result*.json s3://test-result-flask-app'
                }
            }
        }
// //         stage('upload to dynamodb'){
// //             steps{
// //                 sh "aws dynamodb execute-statement --statement \"INSERT INTO test-result VALUE { \'user':\'$BUILD_USER\',\'date\':\'$TIME\',\'state\':\'$RESULT\'}\""
// //             }
// //         }
    }
    post {
        always {
            deleteDir()
        }
    }
}
