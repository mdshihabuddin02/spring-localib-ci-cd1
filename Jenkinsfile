pipeline {
    agent any

    environment {
                DOCKER_REGISTRY = 'https://index.docker.io/v1/'
                DOCKER_IMAGE_NAME = 'mdshihabuddin/locallibrary'
                DOCKER_IMAGE_TAG = '1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'br1', credentialsId: 'git-credential', url: 'https://github.com/mdshihabuddin10/pro-spring1'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    def imageTag = "${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                        def dockerImage = docker.build(imageTag, '-f Dockerfile .')

                        withDockerRegistry(credentialsId: 'docker-cred') {
                            dockerImage.push(DOCKER_IMAGE_TAG)
                        }
                }
            }
        }
        stage('Deploy stage') {
            steps {
                script {
                    kubeconfig(credentialsId: 'kubeconf' ) {
                    sh "sed -i 's/TAG/${DOCKER_IMAGE_TAG}/g' app-deployment.yml" 
                    sh "/usr/bin/kubectl apply -f app-deployment.yml"
                    sh "/usr/bin/kubectl apply -f app-svc.yml"
                }
                }
            }
        }
    }
}