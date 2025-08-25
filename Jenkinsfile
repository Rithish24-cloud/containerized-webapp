pipeline {
    agent any

    environment {
        REGISTRY = "rithishkumar2405"
        IMAGE = "flask-app"
        TAG = "${BUILD_NUMBER}"
        DOCKERHUB = credentials('dockerhub-pass') // DockerHub creds in Jenkins
        KUBECONFIG = "/etc/rancher/k3s/k3s.yaml"  // Mounted K3s config
    }

    parameters {
        booleanParam(name: 'APPLY_MANIFESTS', defaultValue: false, description: 'Apply Kubernetes manifests if first time deploy')
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
                  k3s kubectl apply -f k8s/deployment.yaml
                  k3s kubectl apply -f k8s/service.yaml
                  k3s kubectl apply -f k8s/hpa.yaml
                '''
            }
        }

        stage('Rolling Update') {
            steps {
                sh '''
                  k3s kubectl set image deployment/flask-app flask-container=$REGISTRY/$IMAGE:$TAG --record
                  k3s kubectl rollout status deployment/flask-app
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

