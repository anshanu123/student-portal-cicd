pipeline {
    agent any

    environment {
        APP_NAME = 'student-portal'
        CLUSTER_NAME = 'student-lab'
        K8S_NAMESPACE = 'default'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout and Validate') {
            steps {
                sh '''
                    test -f index.html
                    test -f Dockerfile
                    echo "Application files verified"
                    grep -n "Version:" index.html || true
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    IMAGE_TAG="build-${BUILD_NUMBER}"
                    docker build -t ${APP_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Load Image into kind') {
            steps {
                sh '''
                    IMAGE_TAG="build-${BUILD_NUMBER}"
                    kind load docker-image \
                      ${APP_NAME}:${IMAGE_TAG} \
                      --name ${CLUSTER_NAME}
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    IMAGE_TAG="build-${BUILD_NUMBER}"

                    kubectl set image \
                      deployment/${APP_NAME} \
                      ${APP_NAME}=${APP_NAME}:${IMAGE_TAG} \
                      -n ${K8S_NAMESPACE}

                    kubectl rollout status \
                      deployment/${APP_NAME} \
                      -n ${K8S_NAMESPACE} \
                      --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get deployment ${APP_NAME} \
                      -n ${K8S_NAMESPACE}

                    kubectl get pods \
                      -l app=${APP_NAME} \
                      -n ${K8S_NAMESPACE}

                    kubectl get service ${APP_NAME} \
                      -n ${K8S_NAMESPACE}
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: New image deployed to local Kubernetes.'
        }
        failure {
            echo 'FAILED: Check Console Output for the first error.'
        }
    }
}
