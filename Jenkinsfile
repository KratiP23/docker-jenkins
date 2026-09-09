pipeline {

    agent {
        label 'linux-agent'
    }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['stage', 'prod'],
            description: 'Select the deployment environment'
        )
    }

    environment {
        DOCKER_IMAGE = 'krati07/jenkins-docker-demo'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Show Environment') {
            steps {
                echo "================================="
                echo "Selected Environment: ${params.ENVIRONMENT}"
                echo "Docker Image: ${env.DOCKER_IMAGE}"
                echo "Build Number: ${env.BUILD_NUMBER}"
                echo "================================="
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "================================="
                    echo "Building Docker image"
                    echo "================================="

                    docker build \
                        -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                        .

                    echo "Docker image built successfully."

                    docker images ${DOCKER_IMAGE}
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
                        echo "Logging in to Docker Hub..."

                        echo "$DOCKER_PASSWORD" | docker login \
                            --username "$DOCKER_USERNAME" \
                            --password-stdin

                        echo "Docker Hub login successful."
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {

                script {

                    if (params.ENVIRONMENT == 'stage') {

                        echo "================================="
                        echo "STAGE ENVIRONMENT"
                        echo "No approval required."
                        echo "Pushing image automatically..."
                        echo "================================="

                        sh '''
                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        '''

                        echo "================================="
                        echo "Stage image pushed successfully."
                        echo "Image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                        echo "================================="

                    } else if (params.ENVIRONMENT == 'prod') {

                        echo "================================="
                        echo "PRODUCTION ENVIRONMENT"
                        echo "Approval is required."
                        echo "================================="

                        input(
                            message: "Approve production image ${env.BUILD_NUMBER}?",
                            ok: "Approve and Push",
                            cancel: "Reject"
                        )

                        echo "Production deployment approved."

                        sh '''
                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        '''

                        echo "================================="
                        echo "Production image pushed successfully."
                        echo "Image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                        echo "================================="

                    } else {

                        error(
                            "Invalid environment selected: ${params.ENVIRONMENT}"
                        )
                    }
                }
            }
        }
    }

    post {

        success {
            echo "================================="
            echo "PIPELINE SUCCESS"
            echo "Environment: ${params.ENVIRONMENT}"
            echo "Image: ${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}"
            echo "================================="
        }

        failure {
            echo "================================="
            echo "PIPELINE FAILED"
            echo "Environment: ${params.ENVIRONMENT}"
            echo "================================="
        }
    }
}