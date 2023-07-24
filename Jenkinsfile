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

        stage('Vault') {
            steps {
                withVault(configuration: [timeout: 60, vaultCredentialId: 'vault1', vaultUrl: 'http://192.168.0.168:8200'], vaultSecrets: [[path: 'kv/hello', secretValues: [[isRequired: false, vaultKey: 'springdbuser'], [isRequired: false, vaultKey: 'springdbpass'], [isRequired: false, vaultKey: 'kubconstring']]]]) {
                    // some block
                    sh "sed -i 's|MYCON|${kubconstring}|g' src/main/resources/application.properties"
                    sh "sed -i 's/MYDBUSER/${springdbuser}/g' src/main/resources/application.properties"
                    sh "sed -i 's/MYDBPASS/${springdbpass}/g' src/main/resources/application.properties"
                }
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
                sh 'mv target/*.jar target/app.jar'
            }
        }

        // stage('AV Scan') {
        //     steps {
        //             sh '/usr/bin/clamscan -irf .'
        //     }
        // }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv(installationName: 'sona1') {
                    // Configure the SonarQube analysis properties
                    script {
                        def scannerHome = tool 'SonarScanner'
                        //sh "${scannerHome}/bin/sonar-scanner"
                        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=pro-spring1 -Dsonar.projectName='pro-spring1'"
    }
  }
}

                    }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    dockerImage = docker.build(imageTag, '-f Dockerfile .')
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    sh "sudo /home/shi/tools/trivy1/trivy image ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                        withDockerRegistry(credentialsId: 'docker-cred') {
                            dockerImage.push(DOCKER_IMAGE_TAG)
                        }
                }
            }
        }

        stage('Deploy stage') {
            steps {
                script {
                        sh "sed -i 's/TAG/${DOCKER_IMAGE_TAG}/g' conf/app-deployment.yml"
                        sh '/usr/local/bin/kubectl apply -f conf/app-deployment.yml'
                        sh '/usr/local/bin/kubectl apply -f conf/app-svc.yml'
                }
            }
        }
    }
}
