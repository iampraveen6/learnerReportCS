pipeline {
    agent any

    environment {
        DOCKER_IMAGE_BACKEND = "your-dockerhub-username/backend"
        DOCKER_IMAGE_FRONTEND = "your-dockerhub-username/frontend"
        REGISTRY_CREDENTIALS = 'docker-hub-credentials'
    }

    stages {
        stage('Clone Repositories') {
            steps {
                git url: 'https://github.com/UnpredictablePrashant/learnerReportCS_backend', branch: 'main'
                dir('frontend') {
                    git url: 'https://github.com/UnpredictablePrashant/learnerReportCS_frontend', branch: 'main'
                }
            }
        }

        stage('Build and Push Images') {
            steps {
                script {
                    docker.withRegistry('', REGISTRY_CREDENTIALS) {
                        def backendImage = docker.build("${DOCKER_IMAGE_BACKEND}", '.')
                        backendImage.push('latest')

                        dir('frontend') {
                            def frontendImage = docker.build("${DOCKER_IMAGE_FRONTEND}", '.')
                            frontendImage.push('latest')
                        }
                    }
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                helm upgrade --install learner-report ./helm-chart \
                  --set backend.image=${DOCKER_IMAGE_BACKEND}:latest \
                  --set frontend.image=${DOCKER_IMAGE_FRONTEND}:latest
                '''
            }
        }
    }
}
