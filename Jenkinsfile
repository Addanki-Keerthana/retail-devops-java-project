pipeline {
  agent any

  options {
    skipDefaultCheckout(true)
  }

  environment {
    DOCKERHUB_USER = "Keerthana"
    IMAGE_NAME = "abctechnologies"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build WAR (Maven in Docker)') {
      steps {
        sh '''
          echo "=== Workspace path ==="
          pwd

          echo "=== List files in workspace ==="
          ls -la

          echo "=== Check pom.xml ==="
          test -f pom.xml && echo "pom.xml found ✅" || (echo "pom.xml NOT found ❌" && exit 1)

          docker run --rm \
            -v "$(pwd)":/app -w /app \
            maven:3.9.6-eclipse-temurin-17 \
            mvn clean test package

          echo "=== WAR produced ==="
          ls -la target/*.war
        '''
      }
    }

    stage('Docker Build') {
      steps {
        sh '''
          docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} .
          docker tag ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
        '''
      }
    }

    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DU', passwordVariable: 'DP')]) {
          sh '''
            echo "$DP" | docker login -u "$DU" --password-stdin
            docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
            docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
          '''
        }
      }
    }
  }
}
