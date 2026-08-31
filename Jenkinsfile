pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '247842831814.dkr.ecr.us-east-1.amazonaws.com/maven-app'
        IMAGE_TAG = "dev-${BUILD_NUMBER}"
    }

    tools {
        maven 'Maven'
        jdk 'JDK17'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/Healthcare-devops/Application.git'
            }
        }

        stage('Build') {
            steps {
                echo "Building Maven project..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Unit Tests') {
            steps {
                echo "Running unit tests..."
                sh 'mvn test'
            }
        }

        stage('SonarQube Scan') {
            steps {
                echo "Scanning code with SonarQube..."
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t maven-app:${IMAGE_TAG} ."
            }
        }

        stage('Push to ECR') {
            steps {
                echo "Pushing Docker image to AWS ECR..."

                sh """
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin ${ECR_REPO.split('/')[0]}

                docker tag maven-app:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                docker push ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed. Check the logs above."
        }

        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
    }
}
