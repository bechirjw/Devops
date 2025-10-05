pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')  // 🔑 ID créé dans Jenkins
        DOCKERHUB_REPO = 'jouini926/devops'                      // 🔁 ton repo DockerHub
        IMAGE_TAG = "${GIT_COMMIT.take(7)}"                              // 🔖 tag basé sur le commit
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/bechirjw/Devops.git'
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
                    echo "📦 Pushing image to DockerHub..."
                    sh """
                    echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                    docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
                    docker push ${DOCKERHUB_REPO}:latest
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build & push completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed."
        }
    }
}
