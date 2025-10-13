pipeline {
  agent any

  environment {
    DOCKERHUB_REPO = 'jouini926/devops'
    IMAGE_TAG      = "${GIT_COMMIT.take(7)}"
    SONAR_HOST_URL = 'http://localhost:9000' // ajuste si besoin
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/bechirjw/Devops.git'
      }
    }

    stage('sonarQube Analysis') {
      steps {
        script {
          withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {

              sh '''
                set -e
                mvn clean verify sonar:sonar \
                  -DskipTests=true -DskipITs=true \
                  -Dsonar.host.url=$SONAR_HOST_URL \
                  -Dsonar.login=$SONAR_TOKEN
              '''
            
          }
        }
      }
    }


    stage('Build Docker Image') {
      steps {
        sh """
          echo "🔨 Building Docker image..."
          docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} .
          docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest
        """
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                          usernameVariable: 'DOCKERHUB_USER',
                                          passwordVariable: 'DOCKERHUB_PASS')]) {
          sh '''
            set -e
            echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
            docker push "${DOCKERHUB_REPO}:${IMAGE_TAG}"
            docker push "${DOCKERHUB_REPO}:latest"
            docker logout
          '''
        }
      }
    }
  }

  post {
    success { echo "✅ Build, Sonar & Push: OK" }
    failure { echo "❌ Pipeline failed." }
  }
}
