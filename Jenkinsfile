pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'nessrine80' 
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub-credentials'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git 'https://github.com/nessrine80/TPjenkins-datascientest.git'
            }
        }

        stage('Build des images Docker') {
            steps {
                sh 'docker build -t $DOCKERHUB_USERNAME/movie_service:$IMAGE_TAG ./movie-service'
                sh 'docker build -t $DOCKERHUB_USERNAME/cast_service:$IMAGE_TAG ./cast-service'
            }
        }

        stage('Push vers DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKERHUB_USERNAME/movie_service:$IMAGE_TAG'
                    sh 'docker push $DOCKERHUB_USERNAME/cast_service:$IMAGE_TAG'
                }
            }
        }

        stage('Déploiement vers Kubernetes (hors prod)') {
            when {
                not {
                    branch 'master'
                }
            }
            steps {
                sh '''
                helm upgrade --install movie-app ./movie-app \
                  --namespace dev \
                  --set movieService.image=$DOCKERHUB_USERNAME/movie_service:$IMAGE_TAG \
                  --set castService.image=$DOCKERHUB_USERNAME/cast_service:$IMAGE_TAG
                '''
            }
        }

        stage('Déploiement production (avec validation)') {
            when {
                branch 'master'
            }
            steps {
                input message: "Déployer en production ?", ok: "Déployer"
                sh '''
                helm upgrade --install movie-app ./movie-app \
                  --namespace prod \
                  --set movieService.image=$DOCKERHUB_USERNAME/movie_service:$IMAGE_TAG \
                  --set castService.image=$DOCKERHUB_USERNAME/cast_service:$IMAGE_TAG
                '''
            }
        }
    }
}
