pipeline {
    agent {
        label 'docker-agent-node'
    }
    environment {
        AWS_CREDENTIALS = credentials('aws-ecr-credentials') // Use the ID you created in step 2
        AWS_REGION = 'ap-south-1' // Your region from the screenshot
        AWS_ACCOUNT_ID = '541645813745' // Your account ID from the screenshot
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ECR_REPOSITORY = "simaak/ecr-test"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
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
        }
        stage('Install AWS CLI') {
            steps {
                sh '''
                    if ! command -v aws; then
                      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
                      unzip awscliv2.zip
                      sudo ./aws/install || ./aws/install --install-dir ~/.local/aws-cli --bin-dir ~/.local/bin
                    fi
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${ECR_REPOSITORY}:latest ."
                    // Get the short Git commit hash
                    def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    // Tag the Docker image with the commit hash
                    sh "docker tag ${ECR_REPOSITORY}:latest ${ECR_REPOSITORY}:${commitHash}"
                }
            }
        }
        stage('Login to AWS ECR') {
            steps {
                script {
                    // Configure AWS credentials
                    sh '''
                        export AWS_ACCESS_KEY_ID=${AWS_CREDENTIALS_USR}
                        export AWS_SECRET_ACCESS_KEY=${AWS_CREDENTIALS_PSW}
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    '''
                }
            }
        }
        stage('Tag and Push to ECR') {
            steps {
                script {
                    def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    
                    // Tag images with full ECR path
                    sh "docker tag ${ECR_REPOSITORY}:latest ${IMAGE_NAME}:latest"
                    sh "docker tag ${ECR_REPOSITORY}:${commitHash} ${IMAGE_NAME}:${commitHash}"
                    
                    // Push to ECR
                    sh "docker push ${IMAGE_NAME}:latest"
                    sh "docker push ${IMAGE_NAME}:${commitHash}"
                }
            }
        }
    }
}
