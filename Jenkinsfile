pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'Java17'
    }

    environment {
        SONARQUBE = credentials('sonar-token')  // Create this in Jenkins Credentials
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Nandan365/Devops.git',
                    credentialsId: 'github'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh """
                    mvn sonar:sonar \
                      -Dsonar.projectKey=devops-demo \
                      -Dsonar.host.url=http://<your-sonar-ip>:9000 \
                      -Dsonar.login=$SONARQUBE
                    """
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh """
                docker build -t <ecr_repo>:latest .
                docker push <ecr_repo>:latest
                """
            }
        }
    }
}
