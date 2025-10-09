pipeline {
  agent any

  environment {
    DOCKERHUB_REPO = 'jouini926/devops'
    IMAGE_TAG      = "${GIT_COMMIT.take(7)}"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/bechirjw/Devops.git'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "${tool 'SonarScanner'}/bin/sonar-scanner -Dsonar.verbose=true"
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 10, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
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
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
            sh "docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}"
            sh "docker push ${DOCKERHUB_REPO}:latest"
          }
        }
      }
    }
  }

  post {
    success { echo "✅ Build, Sonar & Push: OK" }
    failure { echo "❌ Pipeline failed." }
  }
}
