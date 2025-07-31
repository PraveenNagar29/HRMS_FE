pipeline {

  agent any

  environment {
    APP = "hrms-frontend"
    IMAGE = "kartik61/hrms-frontend:latest"
  }

  stages {

    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t ${IMAGE} .'
      }
    }

   stage('Trivy Scan') {
  steps {
    sh '''
      mkdir -p trivy-bin
      echo "[INFO] Installing Trivy..."
      curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b trivy-bin

      export PATH=$PATH:$(pwd)/trivy-bin

      echo "[INFO] Running Trivy..."
      ./trivy-bin/trivy image ${IMAGE} --severity CRITICAL,HIGH --format html --output trivy-report.html

      echo "[INFO] Listing files:"
      ls -la
    '''
    archiveArtifacts artifacts: 'trivy-report.html', onlyIfSuccessful: false
    publishHTML(target: [
      reportDir: '.',
      reportFiles: 'trivy-report.html',
      reportName: 'Trivy HTML Report'
    ])
  }
}



    stage('Docker Login & Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'docker-hub-creds',
          usernameVariable: 'DOCKER_USERNAME',
          passwordVariable: 'DOCKER_PASSWORD'
        )]) {
          sh '''
            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
            docker push ${IMAGE}
          '''
        }
      }
    }

    stage('Docker Run') {
      steps {
        sh 'docker stop ${APP} || true && docker rm ${APP} || true'
        sh 'docker run -d --name ${APP} -p 3000:80 ${IMAGE}'
      }
    }
  }

  post {
    failure {
      echo "❌ Pipeline failed. Check console output."
    }
  }
}
