pipeline {
    agent any

    environment {
        IMAGE_NAME = 'flask-jenkins-app'
        CONTAINER_NAME = 'flask-jenkins-container'
    }

    triggers {
        githubPush()
        cron('H 6 * * *')
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning Flask project from GitHub...'
                checkout scm
            }
        }

        stage('Cleanup Old Containers') {
            steps {
                echo 'Stopping and removing old containers...'
                sh '''
                    docker ps -a -q --filter "name=$CONTAINER_NAME" | grep -q . && docker stop $CONTAINER_NAME && docker rm $CONTAINER_NAME || true
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Run Docker Container') {
            steps {
                echo 'Starting new Flask container...'
                sh 'docker run -d -p 5000:5000 --name $CONTAINER_NAME $IMAGE_NAME'
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
            sh '''
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
            '''
            cleanWs()
            echo 'Pipeline completed.'
        }
        failure {
            echo 'Pipeline failed. Check the Jenkins logs for details.'
        }
    }
}
