pipeline {
    agent any

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
                    if (isUnix()) {
                        sh "docker build -t $REGISTRY/$BACKEND_IMAGE:latest ./backend"
                    } else {
                        bat "docker build -t %REGISTRY%/%BACKEND_IMAGE%:latest ./backend"
                    }
                }
            }
        }

        stage('Construire Frontend') {
            steps {
                script {
                    if (isUnix()) {
                        sh "docker build -t $REGISTRY/$FRONTEND_IMAGE:latest ./frontend"
                    } else {
                        bat "docker build -t %REGISTRY%/%FRONTEND_IMAGE%:latest ./frontend"
                    }
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-username', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        if (isUnix()) {
                            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                            sh "docker push $REGISTRY/$BACKEND_IMAGE:latest"
                            sh "docker push $REGISTRY/$FRONTEND_IMAGE:latest"
                        } else {
                            bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                            bat "docker push %REGISTRY%/%BACKEND_IMAGE%:latest"
                            bat "docker push %REGISTRY%/%FRONTEND_IMAGE%:latest"
                        }
                    }
                }
            }
        }

        stage('DÃ©ployer sur Kubernetes') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'kubectl apply -f k8s/'
                    } else {
                        bat 'kubectl apply -f k8s/'
                    }
                }
            }
        }
    }
}

