pipeline {
    agent {
        label 'docker-agent-node'
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('b6d9feb8-e628-48d2-890a-54817dcb2651')
        IMAGE_NAME = "sim33k/notes-backend"
    }

    stages {
        stage('Debug stage') {
            steps {
                sh '''
                    which aws || echo "AWS CLI NOT FOUND"
                    aws --version || true
                    docker --version
                    whoami
                    uname -a
                '''
        }
        stage('Install AWS CLI') {
            steps {
                sh '''
                    if ! command -v aws; then
                      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
                      unzip awscliv2.zip
                      ./aws/install
                    fi
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${IMAGE_NAME}:latest ."

                    // Get the short Git commit hash
                    def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()

                    // Tag the Docker image with the commit hash
                    sh "docker tag ${IMAGE_NAME}:latest ${IMAGE_NAME}:${commitHash}"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub using credentials
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"

                    // Push the latest tag
                    sh "docker push ${IMAGE_NAME}:latest"

                    // Push the commit-hash tag
                    def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    sh "docker push ${IMAGE_NAME}:${commitHash}"
                }
            }
        }
    }
}
