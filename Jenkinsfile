pipeline{
  agent any
    
   
  environment {
        GIT_REPO = 'https://github.com/Nandan365/Jenkinsandjava.git'
        AWS_REGION = 'ap-south-1'
        ECR_REPO_NAME = 'nandan'
        ECR_PUBLIC_REPO_URI = 'public.ecr.aws/s5h4m1m8/nandan'
        IMAGE_TAG = "${BUILD_NUMBER}"//'latest'
        AWS_ACCOUNT_ID = '660376548872' 
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
    }
    
  stages{
        stage('Install_AWS_CLI'){
            steps{
		sh 'whoami'
                  sh '''
		        set -e
                        echo "Installing AWS CLI..."
                        apt update && apt install -y unzip curl

                        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                        rm -rf aws
                        unzip -q awscliv2.zip
                        ./aws/install --update
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
        
        stage("Clone_Repository"){
                 steps{
                         script{
                git url: "${GIT_REPO}", branch: 'main'
            }
                 }
        }
        
        stage('Build') {
            steps {
                script {
                    sh '''
                        echo "Building Java application..."
                        mvn clean -B -Denforcer.skip=true package
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
                        aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/t2k2p9u0
                    '''
                }
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
