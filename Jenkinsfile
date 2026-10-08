pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        DOCKER_IMAGE = 'abhijith006/department-information-portal'
    }

    stages {

        stage('Clone Code') {
            steps {
                echo 'Code is checked out from GitHub.'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .'
                bat 'docker tag %DOCKER_IMAGE%:%BUILD_NUMBER% %DOCKER_IMAGE%:latest'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'dockerhub-token',
                        variable: 'DOCKER_TOKEN'
                    )
                ]) {

                    bat '''
                        powershell -NoProfile -Command "$env:DOCKER_TOKEN | docker login -u 'abhijith006' --password-stdin"
                    '''

                    bat 'docker push %DOCKER_IMAGE%:%BUILD_NUMBER%'
                    bat 'docker push %DOCKER_IMAGE%:latest'

                    bat 'docker logout'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {

                    bat 'kubectl apply -f deployment.yaml'

                    bat 'kubectl set image deployment/department-portal department-portal=%DOCKER_IMAGE%:%BUILD_NUMBER%'

                    bat 'kubectl rollout status deployment/department-portal'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {

                    bat 'kubectl get deployments'
                    bat 'kubectl get pods -o wide'
                    bat 'kubectl get services'
                }
            }
        }
    }
}
