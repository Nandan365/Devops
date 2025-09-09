pipeline {
    agent {
        node {
            label 'Agent'
        }
    }

    environment {
        GIT_REPO = 'https://github.com/Nandan365/Devops.git'
        TOMCAT_CREDENTIALS = 'tomcat-manager-creds'
        TOMCAT_URL = 'http://localhost:8080'
        TOMCAT_PATH = '/usr/share/apache-tomcat'
        ECR_REPO_NAME = 'nandu'
        ECR_PUBLIC_REPO_URI = '660376548872.dkr.ecr.us-east-1.amazonaws.com/nandu'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
    }

    stages {
        stage('Install_AWS_CLI') {
            steps {
                sh '''
                    set -e
                    echo "Installing AWS CLI..."
                    echo "abc" | sudo -S apt update && sudo -S apt install -y unzip curl
                    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                    rm -rf aws
                    unzip -q awscliv2.zip
                    echo "abc" | sudo -S ./aws/install --update
                    aws --version
                '''
            }
        }

        stage('Configure AWS Credentials') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh 'aws sts get-caller-identity'
                }
            }
        }

        stage('Checkout') {
            steps {
                echo "Cloning repo...."
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('Build and Test') {
            steps {
                echo "Building Java application and running tests..."
                sh 'mvn clean package'
            }
            post {
                always {
                    // ✅ Prevents pipeline failure if no test reports are found
                    junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Deploying application to Tomcat. URL: ${TOMCAT_URL}, Path: ${TOMCAT_PATH}"
                    sh 'pwd'
                    sh 'whoami'
                    deploy adapters: [tomcat9(
                        credentialsId: TOMCAT_CREDENTIALS,
                        path: '',
                        url: TOMCAT_URL
                    )],
                    war: 'target/JtProject.war'
                }
            }
        }

        stage('Build_Docker_Image') {
            steps {
                script {
                    sh 'whoami'
                    sh '''
                        echo "Building Docker image..."
                        docker build -t ${IMAGE_URI} .
                    '''
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                        echo "Logging into AWS ECR..."
                        aws ecr get-login-password --region us-east-1 \
                          | docker login --username AWS --password-stdin 660376548872.dkr.ecr.us-east-1.amazonaws.com
                    '''
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh '''
                    echo "Pushing Docker image to ECR..."
                    docker push ${IMAGE_URI}
                '''
            }
        }
    }

    post {
        always {
            echo "Job finished running"
        }
        success {
            echo "job succeeded"
        }
        failure {
            echo "job failed"
        }
    }
}
