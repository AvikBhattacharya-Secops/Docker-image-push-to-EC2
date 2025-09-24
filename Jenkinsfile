pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'yourdockerhubusername/yourapp:latest'
        EC2_USER = 'ec2-user'
        EC2_HOST = '35.154.5.213'
        SSH_KEY = credentials('ec2-ssh-key') // Add your private key to Jenkins credentials
    }

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/your/repo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh """
                            echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                            docker push ${DOCKER_IMAGE}
                        """
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    sh """
                        ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${EC2_USER}@${EC2_HOST} << EOF
                            docker pull ${DOCKER_IMAGE}
                            docker stop myapp || true
                            docker rm myapp || true
                            docker run -d -p 80:5000 --name myapp ${DOCKER_IMAGE}
                        EOF
                    """
                }
            }
        }
    }
}
