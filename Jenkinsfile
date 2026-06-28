pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: docker
      image: docker:25-dind
      securityContext:
        privileged: true
      command: ["dockerd-entrypoint.sh"]
      args: ["--insecure-registry=192.168.1.109:8081"]
'''
        }
    }

    environment {
        HARBOR_REGISTRY = '192.168.1.109:8081'
        IMAGE_NAME       = "${HARBOR_REGISTRY}/library/test-app"
        IMAGE_TAG        = "build-${BUILD_NUMBER}"
    }

    stages {
        stage('Attendre Docker') {
            steps {
                container('docker') {
                    sh "timeout 60 sh -c 'until docker info >/dev/null 2>&1; do sleep 1; done'"
                }
            }
        }

        stage('Build image') {
            steps {
                container('docker') {
                    sh '''
                        cat > Dockerfile <<'EOF'
FROM alpine:3.20
RUN echo "image de test Jenkins -> Harbor" > /info.txt
CMD ["cat", "/info.txt"]
EOF
                        docker build -t $IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        stage('Push vers Harbor') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(credentialsId: 'harbor-creds',
                                                        usernameVariable: 'HARBOR_USER',
                                                        passwordVariable: 'HARBOR_PASS')]) {
                        sh '''
                            echo "$HARBOR_PASS" | docker login $HARBOR_REGISTRY -u "$HARBOR_USER" --password-stdin
                            docker push $IMAGE_NAME:$IMAGE_TAG
                        '''
                    }
                }
            }
        }

        stage('Verifier (pull retour)') {
            steps {
                container('docker') {
                    sh '''
                        docker rmi $IMAGE_NAME:$IMAGE_TAG
                        docker pull $IMAGE_NAME:$IMAGE_TAG
                        docker run --rm $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
    }
}