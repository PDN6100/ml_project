pipeline {
    agent {
        docker {
            image 'myjenkins:latest'   // ton image Jenkins custom
            args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    tools {
        git 'Default'  // Utilisation de l'installation Git que tu as configurée
    }

    environment {
        REGISTRY = "docker.io/pdn6100"
        BACKEND_IMAGE = "ml_project_backend"
        FRONTEND_IMAGE = "ml_project_frontend"
    }

    stages {
        stage('Cloner le repo') {
            steps {
                git branch: 'main', url: 'https://github.com/PDN6100/ml_project.git'
            }
        }

        stage('Construire Backend') {
            steps {
                script {
                    sh """
                        echo 'Construction de l’image backend...'
                        docker build -t $REGISTRY/$BACKEND_IMAGE:latest ./backend
                        docker images
                    """
                }
            }
        }

        stage('Construire Frontend') {
            steps {
                script {
                    sh """
                        echo 'Construction de l’image frontend...'
                        docker build -t $REGISTRY/$FRONTEND_IMAGE:latest ./frontend
                        docker images
                    """
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-username', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo 'Login Docker...'
                            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                            docker push $REGISTRY/$BACKEND_IMAGE:latest
                            docker push $REGISTRY/$FRONTEND_IMAGE:latest
                        """
                    }
                }
            }
        }

        stage('Déployer sur Kubernetes') {
            steps {
                script {
                    sh """
                        echo 'Déploiement sur Kubernetes...'
                        kubectl config view
                        kubectl get nodes
                        kubectl apply -f k8s/
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline terminée.'
            sh 'docker logout || true'
        }
        success {
            echo 'Pipeline réussie !'
        }
        failure {
            echo 'Pipeline échouée ! Vérifie les logs.'
        }
    }
}
