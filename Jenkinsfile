pipeline {
  agent any

  environment {
    GIT_URL       = 'https://github.com/Kanykei9595/Netflix-clone.git'
    GIT_BRANCH    = 'main'
    DOCKER_IMAGE  = 'kanykei0909/netflix-clone:latest'
    DOCKER_CONFIG = "${WORKSPACE}/.docker"
    DOCKER        = '/usr/local/bin/docker'
    KUBECTL       = '/usr/local/bin/kubectl'
  }

  options { timestamps() }

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
          if (!fileExists(env.DOCKER) && fileExists('/opt/homebrew/bin/docker')) {
            env.DOCKER = '/opt/homebrew/bin/docker'
          }
          if (!fileExists(env.KUBECTL) && fileExists('/opt/homebrew/bin/kubectl')) {
            env.KUBECTL = '/opt/homebrew/bin/kubectl'
          }
        }
        sh '''
          echo "Using Docker: $DOCKER"
          $DOCKER --version
        '''
      }
    }

    stage('Docker Login (no helper)') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh '''
            mkdir -p "$DOCKER_CONFIG"
            echo '{}' > "$DOCKER_CONFIG/config.json"
            echo "$PASS" | $DOCKER login -u "$USER" --password-stdin
          '''
        }
      }
    }

   stage('Build Docker Image') {
     steps {
       withCredentials([string(credentialsId: 'tmdb-api-key', variable: 'TMDB_V3_API_KEY')]) {
         sh '''
           echo "Building image with TMDB API key..."
           DOCKER_BUILDKIT=1 $DOCKER build \
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

    stage('Deploy to Kubernetes') {
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
