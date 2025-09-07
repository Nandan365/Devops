pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "123456789012.dkr.ecr.ap-south-1.amazonaws.com/my-app"
    }

    stages {
        stage("Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/Nandan365/Devops.git', credentialsId: 'github'
            }
        }

        stage("Install AWS CLI") {
            steps {
                sh '''
                    echo "Installing AWS CLI..."
                    sudo apt update -y
                    sudo apt install -y unzip curl
                    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                    rm -rf aws
                    unzip -q awscliv2.zip
                    sudo ./aws/install --update
                    aws --version
                '''
            }
        }

        stage("Configure AWS Credentials") {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_Access_Token']]) {
                    sh 'aws sts get-caller-identity'
                }
            }
        }

        stage("Build App") {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('MySonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage("Login to AWS ECR") {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_Access_Token']]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                sh '''
                    docker build -t my-app:latest .
                    docker tag my-app:latest $ECR_REPO:latest
                '''
            }
        }

        stage("Push Docker Image to ECR") {
            steps {
                sh 'docker push $ECR_REPO:latest'
            }
        }
    }

    post {
        always {
            echo "Job finished running"
        }
        failure {
            echo "job failed"
        }
    }
}
