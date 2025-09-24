pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/AvikBhattacharya-Secops/Docker-image-push-to-EC2.git'
        EC2_HOST = '35.154.5.213'                     // Your Ubuntu EC2 instance
        EC2_USER = 'ubuntu'                          // Ubuntu EC2 default SSH user
        SSH_CREDENTIALS_ID = 'ec2'                 // Jenkins stored SSH key ID
        APP_DIR = 'docker_app'                       // App directory on EC2
        IMAGE_NAME = 'avikbhattacharya056/my-local-image'    // Docker image name on EC2
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: "${env.GIT_REPO}"
            }
        }

        stage('Transfer & Deploy to EC2') {
            steps {
                sshagent (credentials: [SSH_CREDENTIALS_ID]) {
                    script {
                        sh """
                        echo "📂 Cleaning old app directory on EC2..."
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} \\
                            'rm -rf ${APP_DIR} && mkdir -p ${APP_DIR}'

                        echo "📦 Transferring application files to EC2..."
                        scp -o StrictHostKeyChecking=no -r * ${EC2_USER}@${EC2_HOST}:${APP_DIR}

                        echo "🐳 Building and running Docker container on EC2..."
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                            cd ${APP_DIR}
                            docker stop myapp || true
                            docker rm myapp || true
                            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                            docker run -d -p 80:5000 --name myapp ${IMAGE_NAME}:${IMAGE_TAG}
EOF
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ App deployed to EC2 (Ubuntu) successfully!'
        }
        failure {
            echo '❌ Deployment failed. Please check the pipeline logs.'
        }
    }
}
