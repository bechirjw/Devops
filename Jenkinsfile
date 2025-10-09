pipeline {
  agent any

  environment {
    DOCKERHUB_REPO = 'jouini926/devops'
    IMAGE_TAG      = "${GIT_COMMIT.take(7)}"
  }

  options {
    timestamps()                                   // keep readable logs
    buildDiscarder(logRotator(numToKeepStr: '20')) // keep last 20 builds
    timeout(time: 30, unit: 'MINUTES')             // global safety timeout
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
          sh """
            ${tool 'SonarScanner'}/bin/sonar-scanner \
              -Dsonar.verbose=true
          """
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
          echo '🔨 Building Docker image...'
          docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} .
          docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest
        """
      }
    }

    stage('Push to DockerHub') {
      when { branch 'main' } // push only from main
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub',
          usernameVariable: 'DOCKERHUB_USER',
          passwordVariable: 'DOCKERHUB_PASS'
        )]) {
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
    always  { sh 'docker image prune -f || true' }
  }
}
