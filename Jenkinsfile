pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/Nandan365/GeminiApp.git'
        TOMCAT_CREDENTIALS = 'tomcat-manager-creds'
        TOMCAT_URL = 'http://localhost:8080'
        TOMCAT_PATH = '/usr/share/apache-tomcat'
        ECR_REPO_NAME = 'nandu'
        ECR_PUBLIC_REPO_URI = '660376548872.dkr.ecr.us-east-1.amazonaws.com/nandu'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
    }

    stages {
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
                    junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Deploying application to Tomcat..."
                    deploy adapters: [tomcat9(
                        credentialsId: TOMCAT_CREDENTIALS,
                        path: '/',
                        url: TOMCAT_URL
                    )],
                    war: 'target/JtProject.war'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t ${IMAGE_URI} .
                '''
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
                sh 'docker push ${IMAGE_URI}'
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
