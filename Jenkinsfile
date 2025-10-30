pipeline {
    agent any

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

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image for new Flask app...'
                sh 'docker build -t flask-jenkins-app .'
            }
        }

        stage('Run Docker Container') {
            steps {
                echo 'Starting new Flask container...'
                sh 'docker run -d -p 5000:5000 --name flask-jenkins-container flask-jenkins-app'
                echo 'Container is running at http://localhost:5000'
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing application response...'
                sh 'sleep 5'
                sh 'curl -f http://localhost:5000 || exit 1'
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
            echo 'Stopping and cleaning old containers...'
            sh 'docker stop flask-jenkins-container || true'
            sh 'docker rm flask-jenkins-container || true'
            cleanWs()
            echo 'Pipeline completed successfully.'
        }
    }
}
