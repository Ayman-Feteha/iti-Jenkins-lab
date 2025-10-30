pipeline {
    agent any

    environment {
        IMAGE_NAME = 'flask-jenkins-app'
        CONTAINER_NAME = 'flask-jenkins-container'
        DOCKER_HOST = 'unix:///var/run/docker.sock'
    }

    triggers {
        githubPush()
        cron('H 6 * * *')
    }

    stages {
        stage('Check Docker Permissions') {
            steps {
                echo 'Checking Docker daemon accessibility...'
                script {
                    try {
                        sh 'docker --version'
                        sh 'docker info'
                        echo 'Docker is accessible!'
                    } catch (Exception e) {
                        echo "Docker permission error detected: ${e.getMessage()}"
                        echo 'Please ensure Jenkins user is added to docker group:'
                        echo 'sudo usermod -aG docker jenkins'
                        echo 'sudo systemctl restart jenkins'
                        error('Docker permissions not configured properly')
                    }
                }
            }
        }

        stage('Clone Repository') {
            steps {
                echo 'Cloning Flask project from GitHub...'
                checkout scm
            }
        }

        stage('Cleanup Old Containers') {
            steps {
                echo 'Stopping and removing old containers...'
                script {
                    try {
                        sh '''
                            docker ps -a -q --filter "name=$CONTAINER_NAME" | grep -q . && docker stop $CONTAINER_NAME && docker rm $CONTAINER_NAME || true
                        '''
                    } catch (Exception e) {
                        echo 'Trying with sudo...'
                        sh '''
                            sudo docker ps -a -q --filter "name=$CONTAINER_NAME" | grep -q . && sudo docker stop $CONTAINER_NAME && sudo docker rm $CONTAINER_NAME || true
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    try {
                        sh 'docker build -t $IMAGE_NAME .'
                    } catch (Exception e) {
                        echo 'Trying with sudo...'
                        sh 'sudo docker build -t $IMAGE_NAME .'
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                echo 'Starting new Flask container...'
                script {
                    try {
                        sh 'docker run -d -p 5000:5000 --name $CONTAINER_NAME $IMAGE_NAME'
                    } catch (Exception e) {
                        echo 'Trying with sudo...'
                        sh 'sudo docker run -d -p 5000:5000 --name $CONTAINER_NAME $IMAGE_NAME'
                    }
                }
                echo 'Container is running at http://localhost:5000'
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing application response...'
                sh '''
                    sleep 5
                    curl -f http://localhost:5000 || (echo "App test failed!" && exit 1)
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Flask application...'
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace and stopping containers...'
            script {
                try {
                    sh '''
                        docker stop $CONTAINER_NAME || true
                        docker rm $CONTAINER_NAME || true
                    '''
                } catch (Exception e) {
                    echo 'Trying cleanup with sudo...'
                    sh '''
                        sudo docker stop $CONTAINER_NAME || true
                        sudo docker rm $CONTAINER_NAME || true
                    '''
                }
            }
            cleanWs()
            echo 'Pipeline completed.'
        }
        failure {
            echo 'Pipeline failed. Check the Jenkins logs for details.'
        }
    }
}
