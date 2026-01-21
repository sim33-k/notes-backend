pipeline {
    agent {
        label 'built-in'
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('sim33k')
        IMAGE_NAME = "sim33k/notes-backend"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/sim33-k/notes-backend.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                    def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    sh "docker tag ${IMAGE_NAME}:latest ${IMAGE_NAME}:${commitHash}"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    sh "docker push ${IMAGE_NAME}:latest"
                    def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    sh "docker push ${IMAGE_NAME}:${commitHash}"
                }
            }
        }
    }
}

