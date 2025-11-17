pipeline {
    agent {
        docker {
            image 'node:18' // Node.js image with npm included
            args '-u root:root' // optional, run as root to avoid permissions issues
        }
    }

    environment {
        IMAGE_NAME = "vamsikrishna212/myapp"
        TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
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
                withCredentials([
                    string(credentialsId: 'dockerhub-username', variable: 'DOCKER_USER'),
                    string(credentialsId: 'dockerhub-password', variable: 'DOCKER_PASS')
                ]) {
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
    }
}
