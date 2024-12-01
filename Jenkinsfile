pipeline {
    environment {
        DOCKER_ID = "julh62"  // Votre identifiant DockerHub
        CAST_SERVICE_IMAGE = "jenkins-devops-exams-cast-service"  // Nom de l'image pour cast-service
        MOVIE_SERVICE_IMAGE = "jenkins-devops-exams-movie-service"  // Nom de l'image pour movie-service
        DOCKER_TAG = "v.${BUILD_ID}.0"  // Tag de l'image basé sur l'ID du build
    }
    agent any  // Utilisation d'un agent Jenkins disponible
    stages {

        // Étape de construction de l'image pour cast_service
        stage('Docker Build Cast Service') {
            steps {
                script {
                    timeout(time: 3, unit: 'MINUTES') {
                        sh '''
                        docker rm -f cast-service-container || true
                        docker build -t $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG -f cast-service/Dockerfile cast-service
                        '''
                    }
                }
            }
        }

        // Étape de construction de l'image pour movie_service
        stage('Docker Build Movie Service') {
            steps {
                script {
                    timeout(time: 3, unit: 'MINUTES') {
                        sh '''
                        docker rm -f movie-service-container || true
                        docker build -t $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG -f movie-service/Dockerfile movie-service
                        '''
                    }
                }
            }
        }

        // Étape de push des images vers DockerHub
        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")  // Récupération du mot de passe DockerHub depuis les credentials Jenkins
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        // Étape de déploiement sur Kubernetes
        stage('Kubernetes Deployment') {
            steps {
                script {
                    // Mise à jour des manifests Kubernetes avec la nouvelle version des images
                          sh """
		sed -i 's#julh62/jenkins-devops-exams-cast-service:.*#julh62/jenkins-devops-exams-cast-service:$DOCKER_TAG#g' cast-db-deployment.yaml
		sed -i 's#julh62/jenkins-devops-exams-movie-service:.*#julh62/jenkins-devops-exams-movie-service:$DOCKER_TAG#g' movie-db-deployment.yaml

                                kubectl replace -f nginx-config.yaml --force
				kubectl apply -f cast-service-config.yaml
				kubectl apply -f cast-db-config.yaml
				kubectl apply -f movie-service-config.yaml
				kubectl apply -f movie-db-config.yaml

				kubectl apply -f cast-db-volumes.yaml	
				kubectl apply -f movie-db-volumes.yaml

				kubectl apply -f cast-db-deployment.yaml
				kubectl apply -f movie-db-deployment.yaml
				kubectl apply -f movie-service-deployment.yaml
				kubectl apply -f cast-service-deployment.yaml
				kubectl apply -f nginx-deployment.yaml

				kubectl apply -f cast-service-service.yaml
				kubectl apply -f movie-service-service.yaml
				kubectl apply -f nginx-service.yaml
				kubectl apply -f cast-db-service.yaml
				kubectl apply -f movie-db-service.yaml
			"""

                }
            }
        }
    }
}
