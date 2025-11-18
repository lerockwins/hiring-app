pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Docker Build') {
            steps {
                sh "docker build . -t sabair0509/hiring-app:${IMAGE_TAG}"
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'hubUser', passwordVariable: 'hubPwd')]) {
                    sh "docker login -u ${hubUser} -p ${hubPwd}"
                    sh "docker push ${hubUser}/hiring-app:${IMAGE_TAG}"
                }
            }
        }

        stage('Checkout K8S manifest SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/betawins/Hiring-app-argocd.git'
            }
        }

        stage('Update K8S manifest & push to Repo') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'Github_server', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) { 
                        sh """
                        sed -i "s/tag: .*/tag: ${IMAGE_TAG}/" dev/deployment.yaml
                        git add dev/deployment.yaml
                        git commit -m 'Updated image tag to ${IMAGE_TAG} via Jenkins Pipeline'
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/betawins/Hiring-app-argocd.git main
                        """
                    }
                }
            }
        }
    }
}
