pipeline {
  agent any

  environment {
    DOCKERHUB_USER = "Keerthana"
    IMAGE_NAME = "abctechnologies"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build WAR (Maven in Docker)') {
      steps {
        sh '''
          docker run --rm \
            -v "$PWD":/app -w /app \
            maven:3.9.6-eclipse-temurin-17 \
            mvn clean test package
        '''
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t $DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER .'
        sh 'docker tag $DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER $DOCKERHUB_USER/$IMAGE_NAME:latest'
      }
    }

    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: "dockerhub-creds", usernameVariable: "DU", passwordVariable: "DP")]) {
          sh 'echo "$DP" | docker login -u "$DU" --password-stdin'
        }
        sh 'docker push $DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER'
        sh 'docker push $DOCKERHUB_USER/$IMAGE_NAME:latest'
      }
    }
  }
}
