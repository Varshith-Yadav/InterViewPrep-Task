pipeline {
    agent any

    environment {
        REMOTE_USER = 'varshithyadav587'
        REMOTE_HOST = '34.57.218.87'
        REMOTE_APP_DIR = '/home/varshithyadav587/flaskapp'
        SSH_KEY = credentials('gcp-vm-ssh') // Use Jenkins credentials (SSH private key)
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Varshith-Yadav/InterViewPrep-Task.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-app .'
            }
        }

        stage('Copy Files to VM') {
            steps {
                sh '''
                rm -rf temp_package
                mkdir temp_package
                cp -r Dockerfile Jenkinsfile app.py requirements.txt temp_package/
                scp -i "$SSH_KEY" -o StrictHostKeyChecking=no -o IdentitiesOnly=yes -r temp_package "$REMOTE_USER@$REMOTE_HOST:$REMOTE_APP_DIR"
                '''
            }
        }

        stage('Deploy on VM') {
            steps {
                sh '''
                ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no -o IdentitiesOnly=yes "$REMOTE_USER@$REMOTE_HOST" << 'EOF'
                cd $REMOTE_APP_DIR/temp_package
                docker stop flask-app || true
                docker rm flask-app || true
                docker build -t flask-app .
                docker run -d -p 80:5000 --name flask-app flask-app
                EOF
                '''
            }
        }
    }
}
