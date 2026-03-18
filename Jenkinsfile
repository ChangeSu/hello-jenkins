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

        stage('Build Image') {
            steps {
                sh 'DOCKER_BUILDKIT=0 docker build -t hello-jenkins:1.0 .'
            }
        }

        stage('List Images') {
            steps {
                sh 'docker images | grep hello-jenkins'
            }
        }

       stage('Run Container') {
            steps {
                sh 'docker rm -f hello-jenkins-demo || true'
                sh 'docker run -d --name hello-jenkins-demo -p 18081:80 hello-jenkins:1.0'
            }
        }

        stage('Verify Service') {
            steps {
                sh 'sleep 10'
                sh 'curl -f http://localhost:18081'
            }
        }
    }
}
