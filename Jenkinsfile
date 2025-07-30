pipeline {
  agent any
 
  environment {
    APP = "hrms-frontend"
    IMAGE = "kartik61/hrms-frontend"
  }
 
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
 
    stage('Docker Build') {
      steps {
        sh 'docker build -t kartik61/hrms-frontend:latest .'
      }
    }
 
    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push  kartik61/hrms-frontend:latest
          '''
        }
      }
    }
 
    stage('Deploy to EC2') {
      steps {
        withCredentials([
          sshUserPrivateKey(credentialsId: 'frontend-ssh', keyFileVariable: 'KEY', usernameVariable: 'SSH_USER'),
          string(credentialsId: 'frontend-ec2-ip', variable: 'REMOTE')
        ]) {
          sh """
            ssh -o StrictHostKeyChecking=no -i $KEY $SSH_USER@$REMOTE '
              docker pull  kartik61/hrms-frontend:latest
              docker stop ${APP} || true
              docker rm ${APP} || true
              docker run -d  -p 80:80  cloudansh/hrms-frontend:latest
            '
          """
        }
      }
    }
  }
}
