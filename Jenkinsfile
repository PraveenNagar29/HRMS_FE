pipeline {
  agent any

  environment {
    APP = "hrms-frontend"
    REMOTE = credentials('frontend-ec2-ip')
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t ${APP}:latest .'
      }
    }

    stage('Trivy Scan') {
      steps {
        sh """
          trivy image --severity LOW,MEDIUM,HIGH --exit-code 0 --format json \
            -o trivy-${APP}.json ${APP}:latest
          trivy image --severity CRITICAL --exit-code 1 --format json \
            -o trivy-${APP}-crit.json ${APP}:latest
          trivy convert --format template \
            --template "@/usr/local/share/trivy/templates/html.tpl" \
            --output trivy-${APP}.html trivy-${APP}.json
        """
      }
      post {
        always {
          publishHTML(target: [
            allowMissing: true,
            alwaysLinkToLastBuild: true,
            reportDir: '.',
            reportFiles: "trivy-${APP}.html",
            reportName: "Trivy Report - ${APP}"
          ])
        }
      }
    }

    stage('Deploy to EC2') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'frontend-ssh', keyFileVariable: 'KEY', usernameVariable: 'SSH_USER')]) {
          sh """
            ssh -o StrictHostKeyChecking=no -i $KEY $SSH_USER@${REMOTE} '
              docker stop ${APP} || true
              docker rm ${APP} || true
              docker run -d --name ${APP} -p 80:80 ${APP}:latest
            '
          """
        }
      }
    }
  }
}
