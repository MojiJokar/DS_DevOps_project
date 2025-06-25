pipeline {
  agent any

  environment {
    DOCKER_ID = "maxjokar2020"
    DOCKER_IMAGE = "ds_devops_project"
    DOCKER_TAG = "v.${BUILD_ID}.0"
  }

  stages {
    stage('Docker Build') {
      steps {
        script {
          dockerImage = docker.build("${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG}")
        }
      }
    }

    stage('Docker Run') {
      steps {
        script {
          sh """
            docker rm -f jenkins || true
            docker run -d -p 80:80 --name jenkins ${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG}
            sleep 10
          """
        }
      }
    }

    stage('Test Acceptance') {
      steps {
        script {
          sh 'curl localhost'
        }
      }
    }

    stage('Docker Push') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'DOCKER_HUB_PASS') {
            dockerImage.push()
          }
        }
      }
    }

    stage('Deploy to Dev') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        script {
          sh """
            rm -Rf .kube && mkdir .kube
            cat \$KUBECONFIG > .kube/config
            cp helm-dev-project/values.yaml values.yml
            sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
            kubectl create namespace dev --dry-run=client -o yaml | kubectl apply -f -
            helm upgrade --install app fastapi --values=values.yml --namespace dev
          """
        }
      }
    }

    stage('Deploy to Staging') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        script {
          sh """
            rm -Rf .kube && mkdir .kube
            cat \$KUBECONFIG > .kube/config
            cp fastapi/values.yaml values.yml
            sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
            kubectl create namespace staging --dry-run=client -o yaml | kubectl apply -f -
            helm upgrade --install app fastapi --values=values.yml --namespace staging
          """
        }
      }
    }

    stage('Deploy to Prod') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        timeout(time: 15, unit: "MINUTES") {
          input message: 'Do you want to deploy in production?', ok: 'Yes'
          script {
            sh """
              rm -Rf .kube && mkdir .kube
              cat \$KUBECONFIG > .kube/config
              cp fastapi/values.yaml values.yml
              sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
              kubectl create namespace prod --dry-run=client -o yaml | kubectl apply -f -
              helm upgrade --install app fastapi --values=values.yml --namespace prod
            """
          }
        }
      }
    }
  }
}
