pipeline {
    agent { label 'k8s-agent' }

    environment {
        IMAGE_NAME = "website"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/tengaleh/beginner-html-site-styled.git'
            }
        }

        stage('Build + Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                        set -e

                        FULL_IMAGE=$DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG

                        echo "Building: $FULL_IMAGE"
                        docker build -t $FULL_IMAGE .

                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push $FULL_IMAGE
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        set -e

                        export KUBECONFIG=$KUBECONFIG

                        kubectl version --client
                        kubectl cluster-info

                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml

                        kubectl rollout restart deployment website
                    '''
                }
            }
        }
    }
}
