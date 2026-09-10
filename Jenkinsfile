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
        GITHUB_REPO = 'KratiP23/docker-jenkins'
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
                                echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
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
                                echo "Triggering GitHub approval workflow..."

                                curl -s -X POST \
                                    -H "Accept: application/vnd.github+json" \
                                    -H "Authorization: Bearer $GITHUB_TOKEN" \
                                    "https://api.github.com/repos/${GITHUB_REPO}/actions/workflows/${GITHUB_WORKFLOW}/dispatches" \
                                    -d '{"ref":"main","inputs":{"image_tag":"'"${BUILD_NUMBER}"'"}}'

                                sleep 10

                                echo "Waiting for approval..."

                                for i in $(seq 1 30); do
                                    RESULT=$(curl -s \
                                        -H "Accept: application/vnd.github+json" \
                                        -H "Authorization: Bearer $GITHUB_TOKEN" \
                                        "https://api.github.com/repos/${GITHUB_REPO}/actions/workflows/${GITHUB_WORKFLOW}/runs?per_page=1")

                                    CONCLUSION=$(echo "$RESULT" | python3 -c "import sys,json; d=json.load(sys.stdin)['workflow_runs']; print(d[0]['conclusion'] or '' if d else '')")

                                    echo "Status check $i: $CONCLUSION"

                                    if [ "$CONCLUSION" = "success" ]; then
                                        echo "Approved!"
                                        exit 0
                                    fi

                                    if [ "$CONCLUSION" = "failure" ] || [ "$CONCLUSION" = "cancelled" ]; then
                                        echo "Rejected."
                                        exit 1
                                    fi

                                    sleep 10
                                done

                                echo "Timed out waiting for approval."
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
                                echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                                docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                                docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:prod
                                docker push ${DOCKER_IMAGE}:prod
                            '''
                        }
                    }
                }
            }
        }
    }
}