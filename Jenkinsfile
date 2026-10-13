
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'vinay0650/newubuntu'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .'
            }
        }

        stage('Docker Hub Login and Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    bat '''
                        @echo off
                        echo %DOCKER_TOKEN% | docker login -u %DOCKER_USER% --password-stdin
                        if errorlevel 1 exit /b 1
                        docker push %IMAGE_NAME%:%BUILD_NUMBER%
                        if errorlevel 1 exit /b 1
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '''
                    kubectl apply -f k8s\\deployment.yaml
                    if errorlevel 1 exit /b 1

                    kubectl apply -f k8s\\service.yaml
                    if errorlevel 1 exit /b 1

                    kubectl set image deployment/tomcatwebapp tomcatwebapp=%IMAGE_NAME%:%BUILD_NUMBER%
                    if errorlevel 1 exit /b 1

                    kubectl rollout status deployment/tomcatwebapp --timeout=180s
                '''
            }
        }
    }
}
