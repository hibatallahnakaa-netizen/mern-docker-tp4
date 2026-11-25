pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build images') {
            steps {
                bat '''
                docker build -t hibatallahnakaa/mern-server:latest ./Server
                if errorlevel 1 exit /b 1
                docker build -t hibatallahnakaa/mern-client:latest ./Client
                if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Deploy') {
            steps {
                bat '''
                docker rm -f client server mongodb 2>nul || ver > nul
                docker network create mern-network 2>nul || ver > nul
                docker run -d --name mongodb --network mern-network mongo
                if errorlevel 1 exit /b 1
                docker run -d --name server --network mern-network -p 9000:9000 -e MONGO_URI=mongodb://mongodb:27017/myapp hibatallahnakaa/mern-server:latest
                if errorlevel 1 exit /b 1
                docker run -d --name client --network mern-network -p 3000:3000 hibatallahnakaa/mern-client:latest
                if errorlevel 1 exit /b 1
                ping -n 8 127.0.0.1 > nul
                docker ps
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                bat 'curl -I http://localhost:3000 | find "200 OK"'
            }
        }

        stage('Push (Docker Hub)') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    bat '''
                    powershell -NoProfile -NonInteractive -Command "$env:DOCKERHUB_TOKEN | docker login -u $env:DOCKERHUB_USER --password-stdin; if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }"
                    if errorlevel 1 exit /b 1
                    docker push hibatallahnakaa/mern-server:latest
                    if errorlevel 1 exit /b 1
                    docker push hibatallahnakaa/mern-client:latest
                    if errorlevel 1 exit /b 1
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Build + Deploy + Push OK'
        }
        failure {
            echo 'Pipeline KO'
        }
    }
}
