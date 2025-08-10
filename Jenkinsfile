pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'iampraveen6'
    IMAGE_BACKEND = "${DOCKERHUB_USER}/backend"
    IMAGE_FRONTEND = "${DOCKERHUB_USER}/frontend"
    DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    KUBECONFIG_CRED = 'kubeconfig-credential-id'
    HELM_RELEASE = "learner-report"
    HELM_CHART_DIR = "learner-report"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/iampraveen6/learnerReportCS.git'
      }
    }

    stage('Build Backend') {
      steps {
        dir('backend') {
          sh "docker build -t ${IMAGE_BACKEND}:${BUILD_NUMBER} ."
        }
      }
    }

    stage('Build Frontend') {
      steps {
        dir('frontend') {
          sh "docker build -t ${IMAGE_FRONTEND}:${BUILD_NUMBER} ."
        }
      }
    }

    stage('Push Images') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh """
            echo "$PASS" | docker login -u "$USER" --password-stdin
            docker push ${IMAGE_BACKEND}:${BUILD_NUMBER}
            docker push ${IMAGE_FRONTEND}:${BUILD_NUMBER}
          """
        }
      }
    }

    stage('Deploy with Helm') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG_FILE')]) {
          sh """
            export KUBECONFIG=${KUBECONFIG_FILE}
            helm upgrade --install ${HELM_RELEASE} ${HELM_CHART_DIR} \
              --set image.backend.repository=${IMAGE_BACKEND} \
              --set image.backend.tag=${BUILD_NUMBER} \
              --set image.frontend.repository=${IMAGE_FRONTEND} \
              --set image.frontend.tag=${BUILD_NUMBER} \
              --namespace default --create-namespace
          """
        }
      }
    }
  }

  post {
    success {
      echo "✅ Deployment successful!"
    }
    failure {
      echo "❌ Deployment failed!"
    }
  }
}
