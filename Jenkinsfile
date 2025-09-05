pipeline {
    agent { label 'Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "660376548872.dkr.ecr.us-east-1.amazonaws.com/devopsnandan"
        IMAGE_TAG = "latest"
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
