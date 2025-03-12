pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'your-dockerhub-username/node-app'
        DOCKER_CREDENTIALS = 'DOCKER_HUB_CREDENTIALS'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: 'GITHUB_CREDENTIALS', url: 'https://github.com/your-repo/node-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['EC2_SSH_CREDENTIALS']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@your-ec2-ip <<EOF
                        docker pull $DOCKER_IMAGE
                        docker stop node-app || true
                        docker rm node-app || true
                        docker run -d --name node-app -p 3000:3000 $DOCKER_IMAGE
                    EOF
                    '''
                }
            }
        }
    }
}
