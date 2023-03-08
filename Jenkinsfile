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
                sh "cp -r app/ ."
              
                sh 'sudo docker build  -t flask_image .'
              sh "sudo docker run -d -p 5000:5000 flask_image"
            }
        }
        stage("build user") {
  steps{
    wrap([$class: 'BuildUser', useGitAuthor: true]) {
      sh 'echo ${BUILD_USER} >> Result.json'
    }
  }
}
      stage("testing") {
    steps {
        script {
           STATUS = sh(script: "curl -I \$(dig +short myip.opendns.com @resolver1.opendns.com):5000 | grep \"HTTP/1.1 200 OK\" | tr -d \"\\r\\n\"", returnStdout: true).trim()
            sh 'echo "$STATUS" >> Result.json'
            sh 'echo "$TIME" >> Result.json'
            withAWS(credentials: 'awscredentials', region: 'us-east-1') {
                sh "aws dynamodb put-item --table-name test-result --item '{\"user\": {\"S\": \"${BUILD_USER}\"}, \"date\": {\"S\": \"${TIME}\"}, \"state\": {\"S\": \"${STATUS}\"}}'"
            }
        }
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
