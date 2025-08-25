pipeline {
  agent any

  environment {
    REGISTRY = "rithishkumar2405"
    IMAGE = "flask-app"
    TAG = "${BUILD_NUMBER}"
    DOCKERHUB = credentials('dockerhub-pass') // set this ID in Jenkins
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Image') {
      steps {
        sh '''
          docker build -t $REGISTRY/$IMAGE:$TAG .
        '''
      }
    }

    stage('Login & Push') {
      steps {
        sh '''
          echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin
          docker push $REGISTRY/$IMAGE:$TAG
        '''
      }
    }

    stage('K8s Apply Manifests (first time)') {
      when { expression { return params.APPLY_MANIFESTS == true } }
      steps {
        sh '''
         sudo k3s kubectl apply -f k8s/deployment.yaml
         sudo k3s kubectl apply -f k8s/service.yaml
         sudo k3s kubectl apply -f k8s/hpa.yaml
        '''
      }
    }

    stage('Rolling Update') {
      steps {
        sh '''
         sudo k3s kubectl set image deployment/flask-app flask-container=$REGISTRY/$IMAGE:$TAG --record
         sudo k3s kubectl rollout status deployment/flask-app
        '''
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
    }
  }
}
