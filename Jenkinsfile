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
        ECR_PUBLIC_REPO_URI = '660376548872.dkr.ecr.us-east-1.amazonaws.com/devopsnandan'
        IMAGE_TAG = "${BUILD_NUMBER}"//'latest'
        AWS_ACCOUNT_ID = '660376548872' 
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Nandan365/Devops.git'
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
                aws ecr get-login-password --region ${AWS_DEFAULT_REGION} \
                | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com
            """
        }
    }
}



        stage("Build Docker Image") {
            steps {
                script {
                    sh """
                    docker build -t devopsnandan:${IMAGE_TAG} .
                    docker tag devopsnandan:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage("Push Image to ECR") {
            steps {
                script {
                    sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
                }
            }
        }
    }
}
