pipeline {
    agent any

    environment {
        DOCKER_ID    = "sabrina2305"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG   = "v.${BUILD_ID}.0"
    }

    stages {

        stage('Building') {
            steps {
                sh '''
                    python3 --version
                    pip3 --version
                    pip3 install --break-system-packages -r requirements.txt
                '''
            }
        }

        stage('Testing') {
            steps {
                sh '''
                    python3 -m unittest
                '''
            }
        }

        stage('User acceptance') {
            steps {
                input message: "Voulez-vous déployer le code ?", ok: "Déployer"
            }
        }

        stage('Deploying') {
            steps {
                sh '''
                    docker rm -f jenkins || true
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                    docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                '''
            }
        }

        stage('Pushing and Merging') {
            parallel {

                stage('Pushing') {
                    environment {
                        DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_PASS')
                    }
                    steps {
                        sh '''
                            echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                            docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                        '''
                    }
                }

                stage('Merging') {
                    steps {
                        echo 'Merging done'
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
