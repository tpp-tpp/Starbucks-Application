pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    stages {
        stage ("Clean Workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/tpp-tpp/Starbucks-Application.git'
            }
        }
        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }
        stage("Build Docker Image") {
            steps {
                sh "docker build -t starbucks ."
            }
        }
        stage("Tag & Push to DockerHub") {
            steps {
                script {

                    withDockerRegistry(credentialsId: 'docker') {

                        sh "docker tag starbucks dadda5/starbucks:${BUILD_NUMBER}"
                        sh "docker push dadda5/starbucks:${BUILD_NUMBER}"

                        sh "docker tag starbucks dadda5/starbucks:latest"
                        sh "docker push dadda5/starbucks:latest"
                    }

                    withCredentials([usernamePassword(
                        credentialsId: 'github',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {

                        sh """
                        rm -rf starbucks-manifests

                        git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/tpp-tpp/cafeday-manifests.git

                        sed -i 's|image: .*|image: dadda5/starbucks:${BUILD_NUMBER}|' cafeday-manifests/k8s/deployment.yaml

                        cd cafeday-manifests

                        git config user.name "Jenkins"
                        git config user.email "jenkins@local"

                        git add k8s/deployment.yaml

                        git commit -m "Update image to ${BUILD_NUMBER}" || true

                        git push origin main
                        """
                    }
                }
            }
        }
    }
}
