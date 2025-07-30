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

          if ! command -v trivy &> /dev/null; then

            echo "Installing Trivy..."

            curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

          fi

          trivy image --severity HIGH,CRITICAL ${IMAGE} || true

        '''

      }

    }
 
    stage('Docker Login & Push') {

      steps {

        withCredentials([usernamePassword(

          credentialsId: 'docker-hub-creds',

          usernameVariable: 'DOCKER_USER',

          passwordVariable: 'DOCKER_PASS'

        )]) {

          sh '''

            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

            docker push ${IMAGE}

          '''

        }

      }

    }

  }
 
  post {

    success {

      echo "✅ Docker image pushed: ${IMAGE}"

    }

    failure {

      echo "❌ Pipeline failed. Check console output."

    }

  }

}
 
