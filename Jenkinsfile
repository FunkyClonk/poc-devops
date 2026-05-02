pipeline {
    agent any

    environment {
        // Remplace avec l'IP de ta VM K3s
        HARBOR_URL     = '192.168.1.103:30002'
        HARBOR_PROJECT = 'poc'
        IMAGE_NAME     = 'poc-devops'
        IMAGE_TAG      = "${BUILD_NUMBER}"
        FULL_IMAGE     = "${HARBOR_URL}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checkout du code depuis GitHub..."
                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo "Lancement des tests..."
                sh 'cd app && npm install && npm test'
            }
        }

        stage('Build Image') {
            steps {
                echo "Build de l image Docker..."
                sh "docker build -t ${FULL_IMAGE} ."
            }
        }

        stage('Push Harbor') {
            steps {
                echo "Push vers Harbor..."
                withCredentials([usernamePassword(
                    credentialsId: 'harbor-credentials',
                    usernameVariable: 'HARBOR_USER',
                    passwordVariable: 'HARBOR_PASS'
                )]) {
                    sh """
                        docker login ${HARBOR_URL} -u ${HARBOR_USER} -p ${HARBOR_PASS}
                        docker push ${FULL_IMAGE}
                        docker logout ${HARBOR_URL}
                    """
                }
            }
        }

        stage('Deploy K3s') {
            steps {
                echo "Déploiement sur K3s..."
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        # Créer le namespace si inexistant
                        kubectl apply -f k8s/namespace.yaml

                        # Créer le secret Harbor si inexistant
                        kubectl create secret docker-registry harbor-secret \
                            --docker-server=${HARBOR_URL} \
                            --docker-username=${env.HARBOR_USER} \
                            --docker-password=${env.HARBOR_PASS} \
                            --namespace=poc \
                            --dry-run=client -o yaml | kubectl apply -f -

                        # Remplacer le tag dans le manifest
                        sed -i 's|IMAGE_TAG|${IMAGE_TAG}|g' k8s/deployment.yaml

                        # Appliquer les manifests
                        kubectl apply -f k8s/service.yaml
                        kubectl apply -f k8s/deployment.yaml

                        # Attendre que le déploiement soit prêt
                        kubectl rollout status deployment/poc-devops -n poc --timeout=120s
                    """
                }
            }
        }

    }

    post {
        success {
            echo "Déploiement réussi ! Image: ${FULL_IMAGE}"
        }
        failure {
            echo "Echec du pipeline !"
        }
        always {
            sh "docker rmi ${FULL_IMAGE} || true"
        }
    }
}
