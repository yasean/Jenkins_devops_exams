pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub_credentials'
        REGISTRY = "docker.io/yhniche"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/yasean/Jenkins_devops_exams.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker-compose build'
                    sh "docker tag jenkins_devops_exams_movie_service:latest $REGISTRY/jenkins_devops_exams_movie_service:$IMAGE_TAG"
                    sh "docker tag jenkins_devops_exams_cast_service:latest $REGISTRY/jenkins_devops_exams_cast_service:$IMAGE_TAG"
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push $REGISTRY/jenkins_devops_exams_movie_service:$IMAGE_TAG
                            docker push $REGISTRY/jenkins_devops_exams_cast_service:$IMAGE_TAG
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes - dev') {
            steps {
                script {
                    sh 'pwd && ls -lah'
                    sh 'cd $WORKSPACE && kubectl apply -v 9 -f k8s/movie-db-deployment.yaml'
                    sh 'cd $WORKSPACE && kubectl apply -v 9 -f k8s/movie-service-deployment.yaml'
                    sh 'cd $WORKSPACE && kubectl apply -v 9 -f k8s/cast-service-deployment.yaml'
                    sh 'cd $WORKSPACE && kubectl apply -v 9 -f k8s/nginx-deployment.yaml'
                }
            }
        }

        stage('Deploy to Kubernetes - prod (manual)') {
            when {
                branch 'master'
            }
            steps {
                script {
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
