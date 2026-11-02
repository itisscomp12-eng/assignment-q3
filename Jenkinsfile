pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "itisscomp12"
        IMAGE_NAME     = "my-java-app"
        IMAGE_TAG      = "${BUILD_NUMBER}"
        FULL_IMAGE     = "${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
    }

    stages {
        stage('Pull Code') {
            steps {
                echo 'Pulling code from GitHub...'
                checkout scm
            }
        }

        stage('Compile Java') {
            steps {
                sh 'javac Hello.java'
            }
        }

        stage('Execute Application') {
            steps {
                sh 'java Hello'
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Run Container') {
            steps {
                sh "docker run --rm ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Tag Image') {
            steps {
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE}"
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push ${FULL_IMAGE}"
                    sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying ${FULL_IMAGE} to Kubernetes"
                sh "sed -i 's|IMAGE_PLACEHOLDER|${FULL_IMAGE}|' k8s/deployment.yaml"
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }

        stage('Verify Pod') {
            steps {
                sh 'kubectl rollout status deployment/my-java-app --timeout=120s'
                sh 'kubectl get pods -l app=my-java-app'
                sh 'kubectl get svc my-java-app-service'
                sh 'kubectl logs -l app=my-java-app'
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs.'
        }
    }
}
