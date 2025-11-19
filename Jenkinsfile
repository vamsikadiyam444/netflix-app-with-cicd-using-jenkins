pipeline {
    agent any

    environment {
        IMAGE_NAME = "vamsikrishna212/myapp"
        TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Install & Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }

        stage('Docker Push') {
            steps {
                // Use Docker Hub credentials stored in Jenkins
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:${TAG}
                    """
                }
            }
        }

        stage('Update Deployment YAML') {
            steps {
                sh """
                    sed -i 's|image:.*|image: ${IMAGE_NAME}:${TAG}|' deployment.yml
                """
            }
        }

        stage('Deployment') {
            steps {
                sh 'kubectl apply -f deployment.yml'
                sh 'kubectl apply -f service.yml'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl get pods'
                sh 'kubectl rollout status deployment/netflix-deployment'
            }
        }
    }
}
