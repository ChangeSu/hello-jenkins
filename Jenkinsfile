pipeline {
    agent any

    stages {
        stage('Checkout Confirm') {
            steps {
                echo 'Source code loaded from repository'
            }
        }

        stage('Show Workspace') {
            steps {
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('Verify Files') {
            steps {
                sh 'test -f index.html'
                sh 'test -f Dockerfile'
                sh 'test -f Jenkinsfile'
                echo 'All required files exist'
            }
        }

       stage('Verify Docker') {
            steps {
                sh 'docker --version'
            }
        }

        stage('Check Docker Client Server') {
            steps {
                sh 'docker version'
            }
        }
    }
}
