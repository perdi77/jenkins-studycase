pipeline {
  agent any

  environment {
    IMAGE = "kibojago/jenkins-studycase"
    TAG = "latest"
    DOCKER_CRED = "docker-hub"
    KUBECONFIG_CRED = "kubeconfig-dev"
    NAMESPACE = "default"
    HELM_RELEASE = "casestudy-jenkins1"
  }

  stages {

    stage('Checkout Source Code') {
      steps {
        // pastikan workspace bersih dan benar-benar git repo
        deleteDir()
        echo "📥 Checking out source code..."
        git branch: 'main', url: 'https://github.com/perdi77/jenkins-studycase.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          echo "🛠️ Building image ${IMAGE}:${TAG}..."
          sh """
            docker build -t ${IMAGE}:${TAG} .
          """
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
            echo "🚀 Deploying to Kubernetes via Helm..."
            sh """
              export KUBECONFIG=$KUBE_FILE
              echo "📄 Using kubeconfig from: $KUBE_FILE"
              helm version
              helm upgrade --install ${HELM_RELEASE} ./helm \
                --set image.repository=${IMAGE} \
                --set image.tag=${TAG} \
                --namespace ${NAMESPACE} --create-namespace
            """
          }
        }
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline Sukses: Aplikasi berhasil dideploy ke Kubernetes"
    }
    failure {
      echo "❌ Pipeline Gagal: Cek log untuk mengetahui error"
    }
  }
}
