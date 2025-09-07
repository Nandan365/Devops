pipeline {
    agent { label 'Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        GIT_REPO = 'https://github.com/Nandan365/Devops.git'
        AWS_REGION = 'us-east-1'
        ECR_REPO_NAME = 'devopsnandan'
        ECR_REPO_URI = '660376548872.dkr.ecr.us-east-1.amazonaws.com/devopsnandan'
        IMAGE_TAG = "${BUILD_NUMBER}" // or "latest"
        AWS_ACCOUNT_ID = '660376548872'
        IMAGE_URI = "${ECR_REPO_URI}:${IMAGE_TAG}"
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: "${GIT_REPO}"
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh """
                        echo "Logging into AWS ECR..."
                        aws ecr get-login-password --region ${AWS_REGION} \
                        | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${ECR_REPO_NAME}:${IMAGE_TAG} .
                    docker tag ${ECR_REPO_NAME}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
                    docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}
                """
            }
        }
    }
}
