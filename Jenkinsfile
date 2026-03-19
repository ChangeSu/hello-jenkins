pipeline {
    agent any

    environment {
        REGISTRY = 'crpi-0xkzvan52s59cv9i.cn-shanghai.personal.cr.aliyuncs.com'
        NAMESPACE = 'changesu'
        IMAGE_NAME = 'hello-jenkins'
        IMAGE_TAG = "${BUILD_NUMBER}"
        FULL_IMAGE = "${REGISTRY}/${NAMESPACE}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

  stages {
      stage('Checkout Confirm') {
          steps {
              echo "Building image: ${FULL_IMAGE}"
              sh 'pwd'
              sh 'ls -la'
          }
      }

      stage('Verify Files') {
          steps {
              sh 'test -f index.html'
              sh 'test -f Dockerfile'
              sh 'test -f Jenkinsfile'
              sh 'test -f k8s/deployment.yaml'
              sh 'test -f k8s/service.yaml'
              echo 'All required files exist'
          }
      }

      stage('Verify Docker') {
          steps {
              sh 'docker version'
          }
      }

      stage('Build Image') {
          steps {
              sh 'DOCKER_BUILDKIT=0 docker build -t hello-jenkins:${IMAGE_TAG} .'
              sh 'docker tag hello-jenkins:${IMAGE_TAG} ${FULL_IMAGE}'
          }
      }

      stage('Login ACR') {
          steps {
              withCredentials([usernamePassword(
                  credentialsId: 'aliyun-acr',
                  usernameVariable: 'REG_USER',
                  passwordVariable: 'REG_PASS'
              )]) {
                  sh 'echo "$REG_PASS" | docker login --username "$REG_USER" --password-stdin ${REGISTRY}'
              }
          }
      }

      stage('Push Image') {
          steps {
              sh 'docker push ${FULL_IMAGE}'
          }
      }

      stage('Prepare Manifest') {
          steps {
              sh '''
                  cp k8s/deployment.yaml k8s/deployment-rendered.yaml
                  sed -i "s#IMAGE_PLACEHOLDER#${FULL_IMAGE}#g" k8s/deployment-rendered.yaml
                  cat k8s/deployment-rendered.yaml
              '''
          }
      }

      stage('Deploy To Kubernetes') {
          steps {
              sh 'kubectl apply -f k8s/deployment-rendered.yaml'
              sh 'kubectl apply -f k8s/service.yaml'
          }
      }

      stage('Check Rollout') {
          steps {
              sh 'kubectl rollout status deployment/hello-jenkins --timeout=120s'
              sh 'kubectl get deployment hello-jenkins'
              sh 'kubectl get pods -l app=hello-jenkins -o wide'
              sh 'kubectl get svc hello-jenkins-service'
          }
      }
  }
}
