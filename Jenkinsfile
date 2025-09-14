pipeline {
    agent {
        docker {
            image 'pdn6100/jenkins-kubectl:latest'
            args '-u root:root'  // pour avoir les permissions si besoin
        }
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
                    sh "docker build -t $REGISTRY/$BACKEND_IMAGE:latest ./backend"
                }
            }
        }

        stage('Construire Frontend') {
            steps {
                script {
                    sh "docker build -t $REGISTRY/$FRONTEND_IMAGE:latest ./frontend"
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-username', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push $REGISTRY/$BACKEND_IMAGE:latest"
                        sh "docker push $REGISTRY/$FRONTEND_IMAGE:latest"
                    }
                }
            }
        }

        stage('Deployer sur Kubernetes') {
            steps {
                script {
                    sh "kubectl apply -f k8s/"
                }
            }
        }
    }
}