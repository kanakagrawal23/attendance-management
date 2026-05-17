pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '595365650066.dkr.ecr.ap-south-1.amazonaws.com'
        ECR_REPO = '595365650066.dkr.ecr.ap-south-1.amazonaws.com/attendance-management'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Test') {
            steps {
                bat 'mvn test'
            }
        }

        stage('Build JAR') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t attendance-app:%IMAGE_TAG% ."
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    bat """
                        aws ecr get-login-password --region %AWS_REGION% > ecr-login.txt
                        docker login --username AWS --password-stdin %ECR_REGISTRY% < ecr-login.txt

                        docker tag attendance-app:%IMAGE_TAG% %ECR_REPO%:%IMAGE_TAG%
                        docker tag attendance-app:%IMAGE_TAG% %ECR_REPO%:latest

                        docker push %ECR_REPO%:%IMAGE_TAG%
                        docker push %ECR_REPO%:latest

                        del ecr-login.txt
                    """
                }
            }
        }
    }

    post {
        success {
            echo "========================================="
            echo "BUILD #${BUILD_NUMBER} SUCCEEDED"
            echo "Image pushed: ${ECR_REPO}:${IMAGE_TAG}"
            echo "========================================="
        }
        failure {
            echo "========================================="
            echo "BUILD #${BUILD_NUMBER} FAILED"
            echo "Check console output: ${BUILD_URL}"
            echo "========================================="
        }
        always {
            cleanWs()
        }
    }
}