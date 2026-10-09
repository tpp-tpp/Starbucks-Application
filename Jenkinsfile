pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-southeast-2'
        ECR_REGISTRY = '347179352513.dkr.ecr.ap-southeast-2.amazonaws.com'
        ECR_REPOSITORY = 'starbut'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short=7 HEAD',
                        returnStdout: true
                    ).trim()

                    env.IMAGE_URI =
                        "${ECR_REGISTRY}/${ECR_REPOSITORY}"

                    sh '''
                        docker build --pull \
                          -t ${IMAGE_URI}:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                    aws ecr get-login-password \
                      --region ${AWS_REGION} |
                    docker login \
                      --username AWS \
                      --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh 'docker push ${IMAGE_URI}:${IMAGE_TAG}'
            }
        }
    }

    post {
        success {
            echo "Successfully pushed ${IMAGE_URI}:${IMAGE_TAG}"
        }

        always {
            sh 'docker logout ${ECR_REGISTRY} || true'
        }
    }
}
