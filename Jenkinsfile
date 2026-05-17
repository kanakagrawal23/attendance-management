pipeline {
    agent any

    options {
        withBuildUser()
    }

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO   = '595365650066.dkr.ecr.ap-south-1.amazonaws.com/attendance-management'
        IMAGE_TAG  = "${BUILD_NUMBER}"
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
                bat "docker build -t attendance-app:${IMAGE_TAG} ."
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
                        docker login --username AWS --password-stdin 595365650066.dkr.ecr.ap-south-1.amazonaws.com < ecr-login.txt

                        docker tag attendance-app:%IMAGE_TAG% %ECR_REPO%:%IMAGE_TAG%
                        docker push %ECR_REPO%:%IMAGE_TAG%

                        del ecr-login.txt
                    """
                }
            }
        }
    }

    post {
        success {
            script {
                if (env.BUILD_USER_EMAIL?.trim()) {
                    mail to: "${env.BUILD_USER_EMAIL}",
                         subject: "Build #${BUILD_NUMBER} SUCCESS",
                         body: "Build passed and image pushed to ECR with tag ${BUILD_NUMBER}."
                } else {
                    echo "BUILD_USER_EMAIL is empty, skipping success email."
                }
            }
        }
        failure {
            script {
                if (env.BUILD_USER_EMAIL?.trim()) {
                    mail to: "${env.BUILD_USER_EMAIL}",
                         subject: "Build #${BUILD_NUMBER} FAILED",
                         body: "Build failed. Check Jenkins console: ${BUILD_URL}"
                } else {
                    echo "BUILD_USER_EMAIL is empty, skipping failure email."
                }
            }
        }
        always {
            cleanWs()
        }
    }
}