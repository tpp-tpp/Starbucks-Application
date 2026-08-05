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

                        sh "docker tag coffday dadda5/coffday:${BUILD_NUMBER}"
                        sh "docker push dadda5/coffday:${BUILD_NUMBER}"

                        sh "docker tag coffday dadda5/coffday:latest"
                        sh "docker push dadda5/coffday:latest"
                    }

                    withCredentials([usernamePassword(
                        credentialsId: 'github',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {

                        sh """
                        rm -rf coffeday-manifests

                        git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/BhairaviDH/coffeday-manifests.git

                        sed -i 's|image: .*|image: dadda5/coffday:${BUILD_NUMBER}|' coffeday-manifests/k8s/deployment.yaml

                        cd coffeday-manifests

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
