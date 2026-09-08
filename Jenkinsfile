pipeline {

    agent {
        label 'linux-agent'
    }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['stage', 'prod'],
            description: 'Select the environment'
        )
    }

    environment {
        DOCKER_IMAGE = 'krati07/jenkins-docker-practical'
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
                    echo "Environment: ${ENVIRONMENT}"
                    echo "Workspace:"
                    pwd

                    echo "Files:"
                    ls -la
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                        -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                        .
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            --username "$DOCKER_USERNAME" \
                            --password-stdin
                    '''
                }
            }
        }

        stage('Push to Docker Hub - Stage') {
            when {
                environment name: 'ENVIRONMENT', value: 'stage'
            }

            steps {
                echo 'Stage environment selected.'
                echo 'Pushing image automatically...'

                sh '''
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Production Approval') {
            when {
                environment name: 'ENVIRONMENT', value: 'prod'
            }

            input {
                message "Production deployment requires approval. Push Docker image ${DOCKER_IMAGE}:${BUILD_NUMBER}?"
                ok "Approve and Push"
                submitter "admin"
            }

            steps {
                echo 'Production deployment approved.'
            }
        }

        stage('Push to Docker Hub - Prod') {
            when {
                environment name: 'ENVIRONMENT', value: 'prod'
            }

            steps {
                echo 'Pushing production image...'

                sh '''
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                '''
            }
        }
    }
}