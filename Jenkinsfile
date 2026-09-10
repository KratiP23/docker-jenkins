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
        GITHUB_REPO = 'KratiP23/docker-Jenkins'
        GITHUB_WORKFLOW = 'promote-prod.yml'
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

                    echo "Production environment selected."
                    echo "Pushing build image and requesting GitHub approval..."

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-credentials',
                            usernameVariable: 'DOCKER_USERNAME',
                            passwordVariable: 'DOCKER_PASSWORD'
                        ),
                        string(
                            credentialsId: 'github-actions-token',
                            variable: 'GITHUB_TOKEN'
                        )
                    ]) {

                        sh '''
                            echo "$DOCKER_PASSWORD" | docker login \
                                --username "$DOCKER_USERNAME" \
                                --password-stdin

                            echo "Pushing build image..."

                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}

                            echo "Triggering GitHub Actions..."

                            curl -L \
                                -X POST \
                                -H "Accept: application/vnd.github+json" \
                                -H "Authorization: Bearer $GITHUB_TOKEN" \
                                -H "X-GitHub-Api-Version: 2026-03-10" \
                                https://api.github.com/repos/${GITHUB_REPO}/actions/workflows/${GITHUB_WORKFLOW}/dispatches \
                                -d "{\"ref\":\"main\",\"inputs\":{\"image_tag\":\"${BUILD_NUMBER}\"}}"

                            echo "GitHub production workflow triggered."
                            echo "Waiting for reviewer approval in GitHub."
                        '''
                    }
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