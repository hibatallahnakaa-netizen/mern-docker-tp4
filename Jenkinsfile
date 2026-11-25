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
                bat 'docker compose build'
            }
        }

        stage('Deploy') {
            steps {
                bat '''
                docker rm -f client server mongodb 2>nul || ver > nul
                docker compose up -d
                ping -n 8 127.0.0.1 > nul
                docker compose ps
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
                    docker compose push
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
