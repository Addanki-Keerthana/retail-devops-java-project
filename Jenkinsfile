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
      echo "WORKSPACE is: $WORKSPACE"
      ls -la "$WORKSPACE"
      test -f "$WORKSPACE/pom.xml" && echo "pom.xml found ✅" || (echo "pom.xml NOT found ❌" && exit 1)

      docker run --rm \
        -v jenkins_home:/var/jenkins_home \
        -w "$WORKSPACE" \
        maven:3.9.6-eclipse-temurin-17 \
        mvn clean test package

      echo "WAR output:"
      ls -la "$WORKSPACE"/target/*.war
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
