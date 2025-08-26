pipeline {
    agent any
    
    
    environment {
        BUILD_TAG = "$(BUILD_NUMBER)"
    }
    triggers {
        githubPush()   
    }

    stages {
        stage('Checkout') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-cred', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    git branch: 'main', url: "https://${GIT_USER}:${GIT_PASS}@github.com/syash7202/ArgoCD-2tire-code.git"
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Frontend') {
                    steps {
                        dir('client') {
                            withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh """
                                    docker build -t ${DOCKER_USER}/client-app:${BUILD_TAG} .
                                """
                            }
                        }
                    }
                }
                stage('Backend') {
                    steps {
                        dir('server') {
                            withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                                sh """
                                    docker build -t ${DOCKER_USER}/server-app:${BUILD_TAG} .
                                """
                            }
                        }
                    }
                }
            }
        }

        stage('Push Images to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_USER}/client-app:latest
                        docker push ${DOCKER_USER}/server-app:latest
                        docker logout
                    """
                }
            }
        }
    }
}
