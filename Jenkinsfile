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
 
    stage('Trivy Filesystem Scan') {
      steps {
        sh '''
          echo "🔍 Running Trivy FS scan..."
          mkdir -p trivy-fs-reports
 
          # Optional: check Trivy version
          docker run --rm aquasec/trivy --version
 
          # Plain text report
          docker run --rm -v $(pwd):/project aquasec/trivy fs \
            --severity HIGH,CRITICAL \
            --format table \
            /project > trivy-fs-reports/trivy-fs-report.json || true
 
          # Optional: HTML report using Trivy template
          mkdir -p contrib
          curl -sSL -o contrib/html.tpl https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl
 
          docker run --rm -v $(pwd):/project -v $(pwd)/contrib:/contrib aquasec/trivy fs \
            --severity HIGH,CRITICAL \
            --format template \
            --template "@/contrib/html.tpl" \
            /project > trivy-fs-reports/trivy-fs-report.html || true
        '''
      }
    }
     stage('Sonar Qube Analysis') {
       steps {
         withSonarQubeEnv("Sonar"){
             sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=hrms -Dsonar.projectKey=hrms"
         }
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
