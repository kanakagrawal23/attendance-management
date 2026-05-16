pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO   = '123456789.dkr.ecr.ap-south-1.amazonaws.com/attendance-management'
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t attendance-app:${IMAGE_TAG} ."
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REPO}

                    docker tag attendance-app:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                    docker push ${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        success {
            mail to: "${env.BUILD_USER_EMAIL}",
                 subject: "Build #${BUILD_NUMBER} SUCCESS",
                 body: "Build passed and image pushed to ECR with tag ${BUILD_NUMBER}."
        }
        failure {
            mail to: "${env.BUILD_USER_EMAIL}",
                 subject: "Build #${BUILD_NUMBER} FAILED",
                 body: "Build failed. Check Jenkins console: ${BUILD_URL}"
        }
    }
}