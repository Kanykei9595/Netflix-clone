pipeline {
  agent any

  environment {
    // ---- SETTINGS ----
    GIT_URL       = 'https://github.com/Kanykei9595/Netflix-clone.git'
    GIT_BRANCH    = 'main'
    DOCKER_IMAGE  = 'kanykei0909/netflix-clone:latest'   // Docker Hub'дагы repo/name:tag
    DOCKER_CONFIG = "${WORKSPACE}/.docker"               // Jenkins үчүн өзүнчө docker config
    DOCKER        = '/usr/local/bin/docker'              // default (Intel)
    KUBECTL       = '/usr/local/bin/kubectl'             // керек болсо жаңырт
  }

  options {
    timestamps()
  }

  stages {
    stage('Clone Repository') {
      steps {
        git branch: "${GIT_BRANCH}",
            credentialsId: 'github-credentials',
            url: "${GIT_URL}"
      }
    }

    stage('Init Tool Paths') {
      steps {
        script {
          // Docker жолун авто-табуу (Apple Silicon/Homebrew үчүн)
          if (!fileExists(env.DOCKER)) {
            if (fileExists('/opt/homebrew/bin/docker')) {
              env.DOCKER = '/opt/homebrew/bin/docker'
            }
          }
          // kubectl да ушундай
          if (!fileExists(env.KUBECTL)) {
            if (fileExists('/opt/homebrew/bin/kubectl')) {
              env.KUBECTL = '/opt/homebrew/bin/kubectl'
            }
          }
        }
        sh '''
          echo "PATH=$PATH"
          echo "Using DOCKER at: $DOCKER"
          $DOCKER --version
          if [ -x "$KUBECTL" ]; then $KUBECTL version --client || true; fi
        '''
      }
    }

    stage('Docker Login (no helper)') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh '''
            mkdir -p "$DOCKER_CONFIG"
            # helper'сиз таза config.json
            echo '{}' > "$DOCKER_CONFIG/config.json"
            echo "$PASS" | $DOCKER login -u "$USER" --password-stdin
          '''
        }
      }
    }

   stage('Build Docker Image') {
     steps {
    // TMDB API ключ Jenkins credentials’тен алынат
       withCredentials([string(credentialsId: 'tmdb-api-key', variable: 'TMDB_V3_API_KEY')]) {
         sh '''
           echo "Building Docker image with TMDB API key..."
           $DOCKER build \
             --build-arg TMDB_V3_API_KEY=$TMDB_V3_API_KEY \
             -t $DOCKER_IMAGE .
         '''
       }
     }
   }
    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh '''
            echo "$PASS" | $DOCKER login -u "$USER" --password-stdin
            $DOCKER push $DOCKER_IMAGE
          '''
        }
      }
    }

    stage('Deploy to Kubernetes (if manifests exist)') {
      when {
        expression { return fileExists('Kubernetes/deployment.yml') && fileExists('Kubernetes/service.yml') }
      }
      steps {
        sh '$KUBECTL apply -f Kubernetes/deployment.yml'
        sh '$KUBECTL apply -f Kubernetes/service.yml'
      }
    }
  }

  post {
    always {
      echo "Build finished with status: ${currentBuild.currentResult}"
    }
  }
}
