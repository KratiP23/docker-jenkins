pipeline {

    agent {
        label 'linux-agent'
    }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['stage', 'prod'],
            description: 'Select environment'
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

        stage('Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .'
            }
        }

        stage('Deploy') {
            steps {
                script {

                    if (params.ENVIRONMENT == 'stage') {

                        withCredentials([
                            usernamePassword(
                                credentialsId: 'dockerhub-credentials',
                                usernameVariable: 'DOCKER_USERNAME',
                                passwordVariable: 'DOCKER_PASSWORD'
                            )
                        ]) {
                            sh '''
                                echo "$DOCKER_PASSWORD" | docker login \
                                    -u "$DOCKER_USERNAME" \
                                    --password-stdin

                                docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            '''
                        }

                    } else {

                        withCredentials([
                            string(
                                credentialsId: 'github-actions-token',
                                variable: 'GITHUB_TOKEN'
                            )
                        ]) {

                            sh '''
                                curl -L --fail-with-body \
                                    -X POST \
                                    -H "Accept: application/vnd.github+json" \
                                    -H "Authorization: Bearer $GITHUB_TOKEN" \
                                    -H "X-GitHub-Api-Version: 2026-03-10" \
                                    "https://api.github.com/repos/${GITHUB_REPO}/actions/workflows/${GITHUB_WORKFLOW}/dispatches" \
                                    --data '{"ref":"main","inputs":{"image_tag":"'"${BUILD_NUMBER}"'"}}'
                            '''

                            echo "GitHub approval requested."

                            sleep 5

                            sh '''
                                for i in $(seq 1 60)
                                do
                                    RESULT=$(curl -s \
                                        -H "Authorization: Bearer $GITHUB_TOKEN" \
                                        "https://api.github.com/repos/${GITHUB_REPO}/actions/workflows/${GITHUB_WORKFLOW}/runs?event=workflow_dispatch&per_page=1")

                                    STATUS=$(echo "$RESULT" | grep -o '"status":"[^"]*"' | head -1 | cut -d'"' -f4)
                                    CONCLUSION=$(echo "$RESULT" | grep -o '"conclusion":"[^"]*"' | head -1 | cut -d'"' -f4)

                                    if [ "$CONCLUSION" = "success" ]; then
                                        exit 0
                                    fi

                                    if [ "$CONCLUSION" = "failure" ] || [ "$CONCLUSION" = "cancelled" ]; then
                                        exit 1
                                    fi

                                    sleep 10
                                done

                                exit 1
                            '''
                        }

                        withCredentials([
                            usernamePassword(
                                credentialsId: 'dockerhub-credentials',
                                usernameVariable: 'DOCKER_USERNAME',
                                passwordVariable: 'DOCKER_PASSWORD'
                            )
                        ]) {
                            sh '''
                                echo "$DOCKER_PASSWORD" | docker login \
                                    -u "$DOCKER_USERNAME" \
                                    --password-stdin

                                docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}

                                docker tag \
                                    ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                                    ${DOCKER_IMAGE}:prod

                                docker push ${DOCKER_IMAGE}:prod
                            '''
                        }
                    }
                }
            }
        }
    }
}