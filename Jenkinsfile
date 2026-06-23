pipeline {
    agent any

    environment {
        APP_NAME = "nestjs-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/prashantsharma0807/nestjs_jenkins.git'
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t $APP_NAME .'
            }
        }

        stage('Stop Old Container') {
            steps {
                sh 'docker stop $APP_NAME || true'
                sh 'docker rm $APP_NAME || true'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker run -d -p 3000:3000 --name $APP_NAME $APP_NAME'
            }
        }
    }
}