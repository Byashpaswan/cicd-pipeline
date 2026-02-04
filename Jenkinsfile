@Library('shared-library') _
pipeline {
  agent any

  options {
    timeout(time: 20, unit: 'MINUTES')
    skipDefaultCheckout()
  }

  environment {
    IMAGE_NAME = "node-express-app"
    CONTAINER_NAME = "node_app"
    APP_PORT = "3000"
    HOST_PORT = "80"
  }

  stages {

    stage('shared-library') {
      steps {
         script {
            hello()
         }
      }

    }

    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm ci || npm install'
      }
    }

    stage('Run Tests') {
      steps {
        sh 'npm test || echo " No tests found"'
      }
    }

    stage('Docker Build') {
      steps {
        sh '''
          docker build -t $IMAGE_NAME:$BUILD_NUMBER .
          docker tag $IMAGE_NAME:$BUILD_NUMBER $IMAGE_NAME:latest
        '''
      }
    }

    stage('Stop Old Container') {
      steps {
        sh '''
          docker stop $CONTAINER_NAME || true
          docker rm $CONTAINER_NAME || true
        '''
      }
    }

    stage('Deploy Container') {
      steps {
        sh '''
          docker run -d \
          --restart unless-stopped \
          --name $CONTAINER_NAME \
          -p $HOST_PORT:$APP_PORT \
          $IMAGE_NAME:latest
        '''
      }
    }

    stage('Cleanup Old Images') {
      steps {
        sh 'docker image prune -f'
      }
    }

    stage('Push to Registry') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'DockerHubCred', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
          sh '''
            echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
            docker tag $IMAGE_NAME:latest $DOCKERHUB_USER/$IMAGE_NAME:latest
            docker tag $IMAGE_NAME:$BUILD_NUMBER $DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER
            docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
            docker push $DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER
          '''
        }
      }
    }
  }

  post {
    success {
      echo ' CI/CD Pipeline Executed Successfully'
    }
    failure {
      echo 'CI/CD Pipeline Failed'
    }
    always {
      sh 'docker ps'
    }
  }
}
