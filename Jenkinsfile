pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub_credentials'
        KUBERNETES_CONTEXT = 'k3s'
        REGISTRY = "docker.io/yhniche"
        IMAGE_TAG = "${GIT_COMMIT}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/yasean/Jenkins_devops_exams'
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    // Construction des images Docker
                    sh 'docker-compose build'
                }
            }
        }

#        stage('Push Images to DockerHub') {
#            steps {
#                script {
#                    // Connexion à DockerHub et push des images
#                    docker.withRegistry('https://docker.io', DOCKERHUB_CREDENTIALS) {
#                        sh "docker push ${REGISTRY}/jenkins_devops_exams_movie_service:${IMAGE_TAG}"
#                        sh "docker push ${REGISTRY}/jenkins_devops_exams_cast_service:${IMAGE_TAG}"
#                    }
#                }
#            }
#        }

         stage('Push Docker Images') {
             steps {
                script {
                    docker.withRegistry('https://docker.io', DOCKERHUB_CREDENTIALS) {
                        sh "docker push ${REGISTRY}/movie-service:${IMAGE_TAG}"
                        sh "docker push ${REGISTRY}/cast-service:${IMAGE_TAG}"
                        // Ajoute ici d'autres services à push si nécessaire
                    }
                }
            }
        }

        stage('Deploy to Kubernetes - dev') {
            steps {
                script {
                    // Déploiement dans l'environnement de développement
                    sh 'kubectl apply -f k8s/movie-db-deployment.yaml'
                    sh 'kubectl apply -f k8s/movie-service-deployment.yaml'
                    sh 'kubectl apply -f k8s/cast-service-deployment.yaml'
                    sh 'kubectl apply -f k8s/nginx-deployment.yaml'
                }
            }
        }

        stage('Deploy to Kubernetes - prod (manual)') {
            when {
                branch 'master'
            }
            steps {
                script {
                    // Déploiement manuel en production
                    input message: 'Deploy to production?', ok: 'Deploy'
                    sh 'kubectl apply -f k8s/prod/movie-db-deployment.yaml'
                    sh 'kubectl apply -f k8s/prod/movie-service-deployment.yaml'
                    sh 'kubectl apply -f k8s/prod/cast-service-deployment.yaml'
                    sh 'kubectl apply -f k8s/prod/nginx-deployment.yaml'
                }
            }
        }
    }

    post {
        success {
            echo 'Déploiement réussi!'
        }
        failure {
            echo 'Erreur dans le pipeline'
        }
    }
}
