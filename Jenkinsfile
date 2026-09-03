
     pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '247842831814.dkr.ecr.us-east-1.amazonaws.com/maven-app'
        IMAGE_TAG = "dev-${BUILD_NUMBER}"
    }

    options {
        ansiColor('xterm')
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/Healthcare-devops/Application.git'
            }
        }

        stage('Maven Build') {
            steps {
                echo 'Building application...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                echo 'Scanning code quality...'
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=maven-app
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh """
                docker build -t maven-app:${IMAGE_TAG} .
                docker tag maven-app:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Push to AWS ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-ecr-credentials'
                ]]) {

                    sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin 247842831814.dkr.ecr.us-east-1.amazonaws.com

                    docker push ${ECR_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }

        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
    }
}
