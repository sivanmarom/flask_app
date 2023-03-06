pipeline {
    agent {label "slave1"}
    
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
        stage ("testing"){
            steps{
           sh 'curl -I $(dig +short myip.opendns.com @resolver1.opendns.com):5000 > Result-${BUILD_USER_FIRST_NAME}-$(date -I).json'
              sh "aws dynamodb batch-write-item --request-items file://${WORKSPACE}/Result*.json"
     
    
                        }
        }
        stage ('upload to s3 bucket'){
            steps{
                withAWS(credentials: 'awscredentials'){
                     sh 'aws s3 cp Result*.json s3://test-result-flask-app'
                }
            }
        }
    }
    post {
        always {
            deleteDir()
        }
    }
}
