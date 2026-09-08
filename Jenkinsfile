pipeline {

    agent {
        label 'linux-agent'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                    echo "Current directory:"
                    pwd

                    echo "Files:"
                    ls -la
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t myapp:1.0 .
                '''
            }
        }

        stage('Verify Docker Image') {
            steps {
                sh '''
                    docker images myapp
                '''
            }
        }

    }
}