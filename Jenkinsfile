pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'rreggejiqnois'
        FRONTEND_IMAGE = "${DOCKER_HUB_USER}/frontend"
        BACKEND_IMAGE  = "${DOCKER_HUB_USER}/backend"
        BUILD_TAG       = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/reppapi/TP2_CLOUD_CICD.git'
            }
        }

        stage('Build & Push') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        // PAKAI "bat" karena ini Windows
                        bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"

                        bat "docker build -t %FRONTEND_IMAGE%:%BUILD_TAG% -t %FRONTEND_IMAGE%:latest ./frontend"
                        bat "docker push %FRONTEND_IMAGE%:%BUILD_TAG%"
                        bat "docker push %FRONTEND_IMAGE%:latest"

                        bat "docker build -t %BACKEND_IMAGE%:%BUILD_TAG% -t %BACKEND_IMAGE%:latest ./backend"
                        bat "docker push %BACKEND_IMAGE%:%BUILD_TAG%"
                        bat "docker push %BACKEND_IMAGE%:latest"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        // PAKAI "bat" dan arahkan kubeconfig-nya
                        bat "kubectl --kubeconfig=%KUBECONFIG% apply -f k8s/backend-deployment.yaml --validate=false"
                        bat "kubectl --kubeconfig=%KUBECONFIG% apply -f k8s/backend-service.yaml --validate=false"
                        bat "kubectl --kubeconfig=%KUBECONFIG% apply -f k8s/frontend-deployment.yaml --validate=false"
                        bat "kubectl --kubeconfig=%KUBECONFIG% apply -f k8s/frontend-service.yaml --validate=false"
                        bat "kubectl --kubeconfig=%KUBECONFIG% apply -f k8s/ingress.yaml --validate=false"

                        bat "kubectl --kubeconfig=%KUBECONFIG% rollout restart deployment/backend-deployment"
                        bat "kubectl --kubeconfig=%KUBECONFIG% rollout restart deployment/frontend-deployment"
                    }
                }
            }
        }
    }
}
