pipeline {

    agent {
        label 'linux-agent'
    }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['stage', 'prod'],
            description: 'Select deployment environment'
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

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Deploy') {
            steps {
                script {

                    if (params.ENVIRONMENT == 'stage') {

                        echo "Deploying to stage..."

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

                                docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            '''
                        }

                    } else if (params.ENVIRONMENT == 'prod') {

                        echo "Preparing production deployment..."

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

                                docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}

                                curl -L --fail-with-body \
                                    -X POST \
                                    -H "Accept: application/vnd.github+json" \
                                    -H "Authorization: Bearer $GITHUB_TOKEN" \
                                    -H "X-GitHub-Api-Version: 2026-03-10" \
                                    "https://api.github.com/repos/${GITHUB_REPO}/actions/workflows/${GITHUB_WORKFLOW}/dispatches" \
                                    --data '{"ref":"main","inputs":{"image_tag":"'"${BUILD_NUMBER}"'"}}'
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "PIPELINE SUCCESS"
            echo "Environment: ${params.ENVIRONMENT}"
            echo "Image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
        }

        failure {
            echo "PIPELINE FAILED"
        }
    }
}