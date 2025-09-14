pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'kubeafreen'
        DOCKER_REPO = 'swayatt-logo-server'
        EC2_HOST = 'ubuntu@54.197.37.108'
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
                    def imageTag = "${DOCKER_REGISTRY}/${DOCKER_REPO}:${env.BUILD_NUMBER}"
                    def latestTag = "${DOCKER_REGISTRY}/${DOCKER_REPO}:latest"

                    sh """
                        docker build -t ${imageTag} .
                        docker tag ${imageTag} ${latestTag}
                    """

                    env.IMAGE_TAG = imageTag
                    env.LATEST_TAG = latestTag
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh """
                        docker push ${env.IMAGE_TAG}
                        docker push ${env.LATEST_TAG}
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${EC2_HOST} '
                            docker login -u ${DOCKER_REGISTRY} -p ${DOCKER_PASS} &&
                            docker pull ${env.LATEST_TAG} &&
                            docker stop myapp || true &&
                            docker rm myapp || true &&
                            docker run -d --name myapp -p 80:3000 ${env.LATEST_TAG}
                        '
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
