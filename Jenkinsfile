pipeline {
    agent any

    environment {
        APP_NAME = 'student-portal'
        IMAGE_TAG = 'jenkins-local'
        CLUSTER_NAME = 'student-lab'
        K8S_NAMESPACE = 'default'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout and Validate') {
            steps {
                sh '''
                    echo "Checking application files..."
                    test -f index.html
                    test -f Dockerfile
                    echo "Validation successful"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${APP_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Load Image into kind') {
            steps {
                sh '''
                    kind load docker-image ${APP_NAME}:${IMAGE_TAG} \
                      --name ${CLUSTER_NAME}
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
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
                    kubectl get deployment ${APP_NAME}
                    kubectl get pods -l app=${APP_NAME}
                    kubectl get service ${APP_NAME}
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Student Portal deployed to local Kubernetes.'
        }
        failure {
            echo 'FAILED: Check the stage logs for the error.'
        }
    }
}
