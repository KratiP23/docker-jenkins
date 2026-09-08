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
        DOCKER_IMAGE = '<YOUR_DOCKER_USERNAME>/jenkins-docker-practical'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                echo "Selected environment: ${params.ENVIRONMENT}"
                echo "Docker image: ${env.DOCKER_IMAGE}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
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
                expression {
                    params.ENVIRONMENT == 'stage'
                }
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
                expression {
                    params.ENVIRONMENT == 'prod'
                }
            }

            input {
                message 'Production deployment requires approval. Continue?'
                ok 'Approve and Push'
            }

            steps {
                echo 'Production deployment approved.'
            }
        }

        stage('Push to Docker Hub - Prod') {

            when {
                expression {
                    params.ENVIRONMENT == 'prod'
                }
            }

            steps {
                echo 'Production approved.'
                echo 'Pushing image to Docker Hub...'

                sh '''
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                '''
            }
        }
    }
}