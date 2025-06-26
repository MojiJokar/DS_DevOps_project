pipeline {
  environment {
    DOCKER_ID = "maxjokar2020" // Your Docker Hub ID
    DOCKER_IMAGE = "datascientestapi"
    DOCKER_TAG = "v.${BUILD_ID}.0"
  }
  agent any

  stages {
    stage('Docker Build') {
      steps {
        script {
          sh '''
          docker rm -f jenkins || true
          docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
          sleep 6
          '''
        }
      }
    }

    stage('Docker Run') {
      steps {
        script {
          sh '''
          docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
          sleep 10
          '''
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
      environment {
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
      }
      steps {
        script {
          sh '''
          docker login -u $DOCKER_ID -p $DOCKER_PASS
          docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
          '''
        }
      }
    }
  } // <-- This closes the 'stages' block
} // <-- This closes the 'pipeline' block
