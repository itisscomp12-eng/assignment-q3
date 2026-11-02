pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "itisscomp12"
    }

    stages {
        stage('Build Image') {
            steps {
                echo "Building image"
                sh "docker build -t java-app ."
            }
        }

        stage('Tag Image') {
            steps {
                echo "Tagging image"
                sh "docker tag java-app ${DOCKERHUB_USER}/java-app5553"
            }
        }

        stage('Push Image') {
            steps {
                echo "Pushing image"
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push ${DOCKERHUB_USER}/java-app5553"
                }
            }
        }
    }
}
