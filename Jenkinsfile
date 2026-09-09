pipeline {

    agent {
        label 'linux-agent'
    }

    environment {
        IMAGE_REPO = 'krati07/jenkins-docker-demo'
        IMAGE_TAG  = "build-${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Docker Check') {
            steps {
                sh '''
                    echo "================================="
                    echo "Docker version"
                    echo "================================="

                    docker --version

                    echo "================================="
                    echo "Docker info"
                    echo "================================="

                    docker info
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "================================="
                    echo "Building Docker image"
                    echo "================================="

                    echo "Repository: ${IMAGE_REPO}"
                    echo "Tag: ${IMAGE_TAG}"

                    docker build \
                        -t ${IMAGE_REPO}:${IMAGE_TAG} \
                        .

                    echo "Docker image built successfully."

                    docker images ${IMAGE_REPO}
                '''
            }
        }

        stage('Docker Login') {
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

        stage('Push Image') {
            steps {
                sh '''
                    echo "================================="
                    echo "Pushing Docker image"
                    echo "================================="

                    echo "Pushing: ${IMAGE_REPO}:${IMAGE_TAG}"

                    docker push ${IMAGE_REPO}:${IMAGE_TAG}

                    echo "================================="
                    echo "IMAGE PUSH SUCCESSFUL"
                    echo "================================="
                '''
            }
        }
    }

    post {

        success {
            echo "================================="
            echo "PIPELINE SUCCESS"
            echo "Image: ${IMAGE_REPO}:${IMAGE_TAG}"
            echo "================================="
        }

        failure {
            echo "================================="
            echo "PIPELINE FAILED"
            echo "================================="
        }
    }
}