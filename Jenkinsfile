pipeline {
    environment { // Declaration of environment variables
        DOCKER_ID = "franciswebandapp" // Replace this with your Docker ID
        DOCKER_IMAGE = "jenkins"
        DOCKER_TAG = "v.${BUILD_ID}.0" // Tag images with the current build ID
    }
    agent any // Jenkins will select all available agents
    stages {
        stage('Docker Build') { // Docker build image stage
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
        stage('Docker Run') { // Run container from the built image
            steps {
                script {
                    sh '''
                    docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    sleep 10
                    '''
                }
            }
        }
        stage('Test Acceptance') { // Validate that the container responds
            steps {
                script {
                    sh '''
                    curl localhost
                    '''
                }
            }
        }
        stage('Docker Push') { // Push the built image to Docker Hub
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // Retrieve Docker password from Jenkins secrets
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
        stage('Deployment in Dev') {
            environment { KUBECONFIG = credentials('config') }
            steps {
                script {
                    sh '''
                    set -eux
                    rm -rf .kube
                    mkdir -p .kube
                    # Write or copy kubeconfig (works for both file and secret-text credentials)
                    if [ -f "$KUBECONFIG" ]; then
                        cp "$KUBECONFIG" .kube/config
                    else
                        printf '%s\\n' "$KUBECONFIG" > .kube/config
                    fi
                    # Debug: surface kubeconfig server and current-context
                    echo "----- kubeconfig (masked) -----"
                    grep -E "server:|current-context:" .kube/config || true
                    echo "----- kubectl version & cluster-info -----"
                    KUBECONFIG=.kube/config kubectl version --client --short || true
                    KUBECONFIG=.kube/config kubectl cluster-info || true
                    echo "----- Try listing namespaces -----"
                    KUBECONFIG=.kube/config kubectl get ns --no-headers || true
                    # Proceed to helm only if kubeconfig appears usable.
                    cp fastapi/values.yaml values.yml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
                    KUBECONFIG=.kube/config helm upgrade --install app fastapi --values=values.yml --namespace dev --create-namespace
                    '''
                }
            }
        }
        stage('Deployment in Staging') {
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
                        printf '%s\\n' "$KUBECONFIG" > .kube/config
                    fi
                    cp fastapi/values.yaml values.yml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
                    KUBECONFIG=.kube/config helm upgrade --install app fastapi --values=values.yml --namespace staging --create-namespace
                    '''
                }
            }
        }
        stage('Deployment to Prod') {
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
                        printf '%s\\n' "$KUBECONFIG" > .kube/config
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