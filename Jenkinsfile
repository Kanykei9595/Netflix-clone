pipeline {
  agent any

  environment {
    GIT_URL = 'https://github.com/Kanykei9595/Netflix-clone.git'
    GIT_BRANCH = 'main'
    DOCKER_IMAGE = 'kanykei0909/netflix-clone:latest'
    DOCKER = '/usr/local/bin/docker'
    KUBECTL = '/usr/local/bin/kubectl'
  }

  stages {
    stage('Clone Repo') {
      steps {
        git branch: "${GIT_BRANCH}",
            credentialsId: 'github-credentials',
            url: "${GIT_URL}"
      }
    }
        
    stage('Docker Login') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh '''
            export DOCKER_CONFIG=$WORKSPACE/.docker
            mkdir -p "$DOCKER_CONFIG"
            echo '{}' > "$DOCKER_CONFIG/config.json"   # helper'сиз таза конфиг
            echo "$PASS" | $DOCKER login -u "$USER" --password-stdin
          '''
        }
      }
    }

    stage('Build & Push') {
      steps {
        sh '''
          export DOCKER_CONFIG=$WORKSPACE/.docker     # ошол эле сессияны колдон
          $DOCKER build -t $DOCKER_IMAGE .
          $DOCKER push $DOCKER_IMAGE
        '''
      }
    }
 
    stage('Deploy to K8s') {
      when {
        expression { fileExists('Kubernetes/deployment.yml') && fileExists('Kubernetes/service.yml') }
      }
      steps {
        sh '''
          export KUBECONFIG=$HOME/.kube/config
          $KUBECTL apply -f Kubernetes/deployment.yml --validate=false
          $KUBECTL apply -f Kubernetes/service.yml --validate=false
        '''
      }
    }
  }
}
