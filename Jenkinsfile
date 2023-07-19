pipeline {
    agent any
        tools {
            maven 'mvn3'
            jdk 'jdk11'
        }

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
        stage("Maven Build"){
            steps{
                sh "mvn clean package"
                sh "mv target/*.war target/app.war"
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
                        sh "sed -i 's/TAG/${DOCKER_IMAGE_TAG}/g' app-deployment.yml"
                        sh '/usr/bin/kubectl apply -f app-deployment.yml'
                        sh '/usr/bin/kubectl apply -f app-svc.yml'
                }
            }
        }
    }
}
