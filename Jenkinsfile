pipeline {
    agent {
        node {
            label 'javadocker'
        }
    }

    environment {
        GIT_REPO = 'https://github.com/Nandan365/Devops.git'
        // Add environment variables for Tomcat credentials and URL
        TOMCAT_CREDENTIALS = 'tomcat-manager-creds'
        TOMCAT_URL = 'http://localhost:8080' // Change this to your Tomcat server's URL and port
        TOMCAT_PATH = '/usr/share/apache-tomcat' // Root context. Use '/yourapp' for a sub-path.
        ECR_REPO_NAME = 'nandan'
        ECR_PUBLIC_REPO_URI = 'public.ecr.aws/s5h4m1m8/nandan'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
    }

    stages {
         stage('Install_AWS_CLI'){
            steps{
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
        
          stage('Configure AWS Credentials'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'AWS_Access_Token', variable: 'AWS_ACCESS_KEY_ID'),
                        string(credentialsId: 'AWS_Secret', variable: 'AWS_SECRET_ACCESS_KEY')]){
                                                   sh '''
                        mkdir -p ~/.aws
                        echo "[default]" | sudo tee ~/.aws/credentials > /dev/null
                        echo "aws_access_key_id=\${AWS_ACCESS_KEY_ID}"
                        echo "aws_secret_access_key=\${AWS_SECRET_ACCESS_KEY}"
                            echo "aws_access_key_id=\${AWS_ACCESS_KEY_ID}" | sudo tee -a ~/.aws/credentials > /dev/null
                            echo "aws_secret_access_key=\${AWS_SECRET_ACCESS_KEY}" | sudo tee -a ~/.aws/credentials > /dev/null
                            
                            # Set correct permissions for the credentials file (only owner can read/write)
                            sudo chmod 600 ~/.aws/credentials
                            
                            echo "AWS credentials file created and secured."
                            ''' 
                        }
                }
            }
        }
        
        stage('Checkout') {
            steps {
                echo "Cloning repo...."
               // checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'GitTokenAdd', url: 'https://github.com/16BitPixel/GeminiApp.git']])
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
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                   //def warFile = "target/*.war"
                    echo "Deploying application to Tomcat. URL: ${TOMCAT_URL}, Path: ${TOMCAT_PATH}"
                    sh ''' pwd '''
                    sh ''' whoami '''
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
        
           stage('Login to AWS ECR'){
             steps {
                script {
                      withCredentials([
                        string(credentialsId: 'AWS_Access_Token', variable: 'AWS_ACCESS_KEY_ID'),
                        string(credentialsId: 'AWS_Secret', variable: 'AWS_SECRET_ACCESS_KEY')
                    ]){
                    
                    sh '''
                        echo "Logging into AWS ECR..."
                        aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/s5h4m1m8/nandan
                    '''
                }
            }

        }
    }
        
         stage('Push Docker Image to ECR') {
            steps {
                script {
                    sh '''
                        echo "Pushing Docker image to ECR..."
                        docker push ${IMAGE_URI}
                    '''
                }
            }
        }
    }
    
     post{
        always{
            echo "Job finished running"
        }
        success{
            echo "job succeeded"
        }
        failure{
            echo "job failed"
        }
    }
}
