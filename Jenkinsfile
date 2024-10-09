pipeline {
    agent any
    tools {
        nodejs 'node16'
    }
    environment {
        registryCredential = 'ecr:us-east-1:awscreds'
        appRegistry = '770934957047.dkr.ecr.us-east-1.amazonaws.com/bingoapp'
        bingoRegistry = 'https://770934957047.dkr.ecr.us-east-1.amazonaws.com'
        cluster = 'bingocluster'
        service = 'bingosvc'
    }
    stages {
        stage('Clean the workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout the project from github repo') {
            steps {
                git branch: 'main', url: 'git@github.com:daus2936/bingo-ecs.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('Build App Image') {
            steps {
                script {
                    dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./dockerfile")
                }
            }
        }
        
        stage('Upload App Image') {
          steps{
            script {
              docker.withRegistry( bingoRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
        }

        stage('Deploy to ECS staging') {
            steps {
                withAWS(credentials: 'awscreds', region: 'us-east-1') {
                    sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
                } 
            }
        }
    }
}