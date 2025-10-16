// ...existing 
pipeline {
  environment {
    DOCKER_ID = "franciswebandapp"
    DOCKER_IMAGE = "jenkins"
    DOCKER_TAG = "v.${BUILD_ID}.0"
  }
  agent any
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Docker Build') {
      steps {
        script {
          sh '''
          set -eux
          docker rm -f jenkins || true
          docker build -t "$DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG" .
          sleep 2
          '''
        }
      }
    }

    stage('Docker Run') {
      steps {
        script {
          sh '''
          set -eux
          docker rm -f jenkins || true
          docker run -d -p 80:80 --name jenkins "$DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG"
          sleep 5
          '''
        }
      }
    }

    stage('Test Acceptance') {
      steps {
        script {
          sh '''
          set -eux
          curl -sSf http://localhost/
          '''
        }
      }
    }

    stage('Docker Push') {
      environment {
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
      }
      steps {
        script {
          sh '''
          set -eux
          echo "$DOCKER_PASS" | docker login -u "$DOCKER_ID" --password-stdin
          docker push "$DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG"
          '''
        }
      }
    }

    stage('Deployment in dev') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        script {
          sh '''
          set -eux
          rm -rf .kube
          mkdir -p .kube
          # if KUBECONFIG is a file path provided by Jenkins credentials, copy; otherwise write content
          if [ -f "$KUBECONFIG" ]; then
            cp "$KUBECONFIG" .kube/config
          else
            printf '%s\n' "$KUBECONFIG" > .kube/config
          fi
          cp fastapi/values.yaml values.yml
          sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
          KUBECONFIG=.kube/config helm upgrade --install app fastapi --values=values.yml --namespace dev --create-namespace
          '''
        }
      }
    }

    stage('Deployment in staging') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        script {
          sh '''
          set -eux
          rm -rf .kube
          mkdir -p .kube
          if [ -f "$KUBECONFIG" ]; then
            cp "$KUBECONFIG" .kube/config
          else
            printf '%s\n' "$KUBECONFIG" > .kube/config
          fi
          cp fastapi/values.yaml values.yml
          sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
          KUBECONFIG=.kube/config helm upgrade --install app fastapi --values=values.yml --namespace staging --create-namespace
          '''
        }
      }
    }

    stage('Deployment to prod') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        timeout(time: 15, unit: 'MINUTES') {
          input message: 'Do you want to deploy in production?', ok: 'Yes'
        }
        script {
          sh '''
          set -eux
          rm -rf .kube
          mkdir -p .kube
          if [ -f "$KUBECONFIG" ]; then
            cp "$KUBECONFIG" .kube/config
          else
            printf '%s\n' "$KUBECONFIG" > .kube/config
          fi
          cp fastapi/values.yaml values.yml
          sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
          KUBECONFIG=.kube/config helm upgrade --install app fastapi --values=values.yml --namespace prod --create-namespace
          '''
        }
      }
    }
  }

  post {
    always {
      script {
        sh '''
        set -eux || true
        docker rm -f jenkins || true
        '''
      }
    }
  }
}
```// filepath: /home/cyriacus1210/datascientest-jenkins/Jenkinsfile
// ...existing code...
pipeline {
  environment {
    DOCKER_ID = "franciswebandapp"
    DOCKER_IMAGE = "jenkins"
    DOCKER_TAG = "v.${BUILD_ID}.0"
  }
  agent any
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Docker Build') {
      steps {
        script {
          sh '''
          set -eux
          docker rm -f jenkins || true
          docker build -t "$DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG" .
          sleep 2
          '''
        }
      }
    }

    stage('Docker Run') {
      steps {
        script {
          sh '''
          set -eux
          docker rm -f jenkins || true
          docker run -d -p 80:80 --name jenkins "$DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG"
          sleep 5
          '''
        }
      }
    }

    stage('Test Acceptance') {
      steps {
        script {
          sh '''
          set -eux
          curl -sSf http://localhost/
          '''
        }
      }
    }

    stage('Docker Push') {
      environment {
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
      }
      steps {
        script {
          sh '''
          set -eux
          echo "$DOCKER_PASS" | docker login -u "$DOCKER_ID" --password-stdin
          docker push "$DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG"
          '''
        }
      }
    }

    stage('Deployment in dev') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        script {
          sh '''
          set -eux
          rm -rf .kube
          mkdir -p .kube
          # if KUBECONFIG is a file path provided by Jenkins credentials, copy; otherwise write content
          if [ -f "$KUBECONFIG" ]; then
            cp "$KUBECONFIG" .kube/config
          else
            printf '%s\n' "$KUBECONFIG" > .kube/config
          fi
          cp fastapi/values.yaml values.yml
          sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
          KUBECONFIG=.kube/config helm upgrade --install app fastapi --values=values.yml --namespace dev --create-namespace
          '''
        }
      }
    }

    stage('Deployment in staging') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        script {
          sh '''
          set -eux
          rm -rf .kube
          mkdir -p .kube
          if [ -f "$KUBECONFIG" ]; then
            cp "$KUBECONFIG" .kube/config
          else
            printf '%s\n' "$KUBECONFIG" > .kube/config
          fi
          cp fastapi/values.yaml values.yml
          sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
          KUBECONFIG=.kube/config helm upgrade --install app fastapi --values=values.yml --namespace staging --create-namespace
          '''
        }
      }
    }

    stage('Deployment to prod') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        timeout(time: 15, unit: 'MINUTES') {
          input message: 'Do you want to deploy in production?', ok: 'Yes'
        }
        script {
          sh '''
          set -eux
          rm -rf .kube
          mkdir -p .kube
          if [ -f "$KUBECONFIG" ]; then
            cp "$KUBECONFIG" .kube/config
          else
            printf '%s\n' "$KUBECONFIG" > .kube/config
          fi
          cp fastapi/values.yaml values.yml
          sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
          KUBECONFIG=.kube/config helm upgrade --install app fastapi --values=values.yml --namespace prod --create-namespace
          '''
        }
      }
    }
  }

  post {
    always {
      script {
        sh '''
        set -eux || true
        docker rm -f jenkins || true
        '''
      }
    }
  }
}