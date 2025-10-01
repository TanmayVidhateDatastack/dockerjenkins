pipeline {
    agent any

    environment {
        COMPOSE_FILE = "dockerjenkins/docker-compose.yml"
        IMAGE_NAME   = "datastackdevops/tender_ui"
    }

    stages {
        stage('Checkout Project Code') {
            steps {
                script {
                    echo "Cloning Project Repository..."
                }
                git branch: 'main',
                    credentialsId: 'Github_token',
                    url: 'https://github.com/TanmayVidhateDatastack/tenderdemo.git'
            }
        }
 stage('Clone Dependencies') {

            steps {

                script {

                    echo "Cloning Elements and Common dependencies..."

                    dir('lib/Elements') {

                        git branch: 'dev',

                            credentialsId: 'Github_token',

                            url: 'https://github.com/Datastack-Technologies/DS-React-Components.git'

                    }

                    dir('lib/Common') {

                        git branch: 'dev',

                            credentialsId: 'Github_token',

                            url: 'https://github.com/Datastack-Technologies/CommonUI.git'

                    }

                }

            }

        }
        stage('Checkout Docker/Compose Repo') {
            steps {
                dir('dockerjenkins') {
                    git branch: 'main',
                        credentialsId: 'Github_token',
                        url: 'https://github.com/TanmayVidhateDatastack/dockerjenkins.git'
                }
            }
        }

        stage('Extract Image Tag') {
            steps {
                script {
                    echo "Extracting Image Tag"
                    def composeFile = readFile(env.COMPOSE_FILE)
                    def imageLine = composeFile.readLines().find { it.contains(env.IMAGE_NAME) }

                    if (!imageLine) {
                        error("Could not find '${env.IMAGE_NAME}' in ${env.COMPOSE_FILE}")
                    }

                    def imageTag = imageLine.substring(imageLine.lastIndexOf(':') + 1).trim()
                    if (!imageTag) {
                        error("Failed to extract IMAGE_TAG from line: ${imageLine}")
                    }

                    env.IMAGE_TAG = imageTag
                    echo "Extracted Image Tag: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image"
                }
                sh "docker-compose -f ${env.COMPOSE_FILE} build"
             
            }
        }

        
       
        
        stage('Deploy with Docker Compose') {
            steps {
                sh "docker-compose -f ${env.COMPOSE_FILE} up -d"
            }
        }

        stage('Cleanup Old Images') {
            steps {
                sh "docker images ${env.IMAGE_NAME} --format '{{.ID}}' | tail -n +2 | xargs -r docker rmi -f || true"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
