pipeline {
  agent any

  environment {
    IMAGE = "kibojago/jenkins-studycase"
    TAG = "latest"
    DOCKER_CRED = "docker-hub"
    GITHUB_CRED = "github-cred"        // <--- tambahkan credential GitHub di Jenkins
    KUBECONFIG_CRED = "kubeconfig-dev"
    NAMESPACE = "default"
    HELM_RELEASE = "casestudy-jenkins1"
  }

  stages {

    stage('Checkout Source Code') {
      steps {
        echo "📥 Checking out source code from GitHub..."
        cleanWs()  // hapus workspace lama biar gak ada sisa
        git branch: 'main',
            credentialsId: "${GITHUB_CRED}",
            url: 'https://github.com/perdi77/jenkins-studycase.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          echo "🛠️ Building Docker image: ${IMAGE}:${TAG}..."
          dockerImage = docker.build("${IMAGE}:${TAG}")
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: "${DOCKER_CRED}",
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          script {
            echo "📦 Pushing image to Docker Hub..."
            sh """
              echo "$PASS" | docker login -u "$USER" --password-stdin
              docker push ${IMAGE}:${TAG}
              docker logout
            """
          }
        }
      }
    }

    stage('Deploy to Kubernetes (Helm)') {
      agent {
        docker {
          image 'alpine/helm:3.14.0'
          args '--entrypoint="" -v /var/jenkins_home/.kube:/root/.kube'
        }
      }
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBE_FILE')]) {
          script {
            echo "🚀 Deploying to Kubernetes using Helm..."
            sh """
              echo '📄 Using kubeconfig from: $KUBE_FILE'
              export KUBECONFIG=$KUBE_FILE

              helm version
              helm upgrade --install ${HELM_RELEASE} ./helm \
                --set image.repository=${IMAGE} \
                --set image.tag=${TAG} \
                --namespace ${NAMESPACE} \
                --create-namespace
            """
          }
        }
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline Sukses: Aplikasi berhasil dideploy ke Kubernetes!"
    }
    failure {
      echo "❌ Pipeline Gagal: Periksa log di tiap stage untuk melihat penyebab error."
    }
  }
}
