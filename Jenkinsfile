pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kanykei9595/netflix-clone"
        
        DOCKER = "/usr/local/bin/docker"   // Intel Mac
        // DOCKER = "/opt/homebrew/bin/docker"  // Apple Silicon (M1/M2)
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/Kanykei9595/Netflix-clone.git'
            }
        }

        stage('Check Docker') {
            steps {
                sh '$DOCKER --version'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '$DOCKER build -t $DOCKER_IMAGE:latest .'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running basic checks...'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                      echo $PASS | $DOCKER login -u $USER --password-stdin
                      $DOCKER push $DOCKER_IMAGE:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when { expression { return fileExists('Kubernetes/deployment.yml') } }
            steps {
                sh 'kubectl apply -f Kubernetes/deployment.yml'
                sh 'kubectl apply -f Kubernetes/service.yml'
            }
        }
    }
}
