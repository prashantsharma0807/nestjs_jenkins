pipeline {
    agent any

    environment {
        APP_NAME = "nestjs-app"
        PORT = "3000"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build --progress=plain -t nestjs-app .
             '''
            }
        }

        stage('Stop Old Container') {
            steps {
                sh '''
                    docker stop $APP_NAME || true
                    docker rm $APP_NAME || true
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker run -d \
                    --name $APP_NAME \
                    -p 3000:3000 \
                    $APP_NAME
                '''
            }
        }

    }
}
