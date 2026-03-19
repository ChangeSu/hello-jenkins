pipeline {
    agent any

    environment {
        REGISTRY = 'crpi-0xkzvan52s59cv9i.cn-shanghai.personal.cr.aliyuncs.com'
        NAMESPACE = 'changesu'
        IMAGE_NAME = 'hello-jenkins'
        IMAGE_TAG = "${BUILD_NUMBER}"
        FULL_IMAGE = "${REGISTRY}/${NAMESPACE}/${IMAGE_NAME}:${IMAGE_TAG}"
        INGRESS_YAML_PATH = 'k8s/ingress.yaml'
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
              sh 'test -f k8s/Jenkinsfile'
              sh 'test -f k8s/deployment.yaml'
              sh 'test -f k8s/service.yaml'
              sh 'test -f ${INGRESS_YAML_PATH}'
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
              sh 'DOCKER_BUILDKIT=0 docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
              sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE}'
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

      stage('Ensure Namespace') {
          steps {
              sh '''
                  kubectl get namespace ${NAMESPACE} || kubectl create namespace ${NAMESPACE}
              '''
          }
      }

      stage('Deploy To Kubernetes') {
          steps {
              sh 'kubectl apply -f k8s/deployment-rendered.yaml -n ${NAMESPACE}'
              sh 'kubectl apply -f k8s/service.yaml -n ${NAMESPACE}'
              sh 'kubectl apply -f ${INGRESS_YAML_PATH} -n ${NAMESPACE} --dry-run=client'
              sh 'kubectl apply -f ${INGRESS_YAML_PATH} -n ${NAMESPACE}'
          }
      }

      stage('Check Rollout') {
          steps {
              sh 'kubectl rollout status deployment/hello-jenkins -n ${NAMESPACE} --timeout=120s'
              sh 'kubectl get deployment hello-jenkins -n ${NAMESPACE}'
              sh 'kubectl get pods -n ${NAMESPACE} -l app=hello-jenkins -o wide'
              sh 'kubectl get svc hello-jenkins-service -n ${NAMESPACE}'

              sh '''
                  kubectl get ingress -n ${NAMESPACE}
                  kubectl describe ingress hello-jenkins-ingress -n ${NAMESPACE}

                  INGRESS_IP=$(kubectl get ingress hello-jenkins-ingress -n ${NAMESPACE} -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
                  INGRESS_HOST=$(kubectl get ingress hello-jenkins-ingress -n ${NAMESPACE} -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

                  if [ -n "$INGRESS_IP" ]; then
                      echo "Ingress部署成功，访问地址：http://${INGRESS_IP}"
                  elif [ -n "$INGRESS_HOST" ]; then
                      echo "Ingress部署成功，访问地址：http://${INGRESS_HOST}"
                  else
                      echo "警告：Ingress暂无外部地址，请检查 Ingress Controller 或 Service 类型"
                  fi
              '''
          }
      }
  }
}
