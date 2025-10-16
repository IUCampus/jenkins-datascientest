pipeline {
  environment { // Declaration of environment variables
    DOCKER_ID = "franciswebandapp" // replace this with your docker-id
    DOCKER_IMAGE = "jenkins"
    DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
  }
  agent any // Jenkins will be able to select all available agents
  stages {
    stage('Docker Build') { // docker build image stage
      steps {
        script {
          sh '''
          docker rm -f jenkins
          docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
          sleep 6
          '''
        }
      }
    }

    stage(' Docker run') { // run container from our built image
      steps {
        script {
          sh '''
          docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
          sleep 10
          '''
        }
      }
    }

    stage('Test Acceptance') { // we launch the curl command to validate that the container responds to the request
      steps {
        script {
          sh '''
          curl localhost
          '''
        }
      }
    }

    stage('Docker Push') { //we pass the built image to our docker hub account
      environment {
            DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve docker password from secret text called docker_hub_pass saved on jenkins
        }
      steps {
        script {
          sh '''
          docker login -u $DOCKER_ID -p $DOCKER_PASS
          docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
          '''
        }
      }
    }

    stage('Deployment in dev') {
      environment { KUBECONFIG = credentials('config') }
      steps {
        script {
          sh '''
          set -eux
          rm -rf .kube
          mkdir -p .kube

          # write or copy kubeconfig (works for both file and secret-text credentials)
          if [ -f "$KUBECONFIG" ]; then
            cp "$KUBECONFIG" .kube/config
          else
            printf '%s\n' "$KUBECONFIG" > .kube/config
          fi

          # debug: surface kubeconfig server and current-context
          echo "----- kubeconfig (masked) -----"
          grep -E "server:|current-context:" .kube/config || true
          echo "----- kubectl version & cluster-info -----"
          KUBECONFIG=.kube/config kubectl version --client --short || true
          KUBECONFIG=.kube/config kubectl cluster-info || true
          echo "----- try listing namespaces -----"
          KUBECONFIG=.kube/config kubectl get ns --no-headers || true

          # If the above commands show HTML or "authentication required", kubeconfig is invalid/auth-less.
          # Proceed to helm only if kubeconfig appears usable.
          cp fastapi/values.yaml values.yml
          sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
          KUBECONFIG=.kube/config helm upgrade --install app fastapi --values=values.yml --namespace dev --create-namespace
          '''
        }
      }
    }

    stage('Deployment in staging') {
      environment { KUBECONFIG = credentials('config') }
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
      environment { KUBECONFIG = credentials('config') }
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
}