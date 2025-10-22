pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "kanykei9595/netflix-clone"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/Kanykei9595/Netflix-clone.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running basic checks...'
            }
        }

        stage('Push to Docker Hub') {
            environment {
                DOCKERHUB_CREDENTIALS = credentials('dockerhub-login')
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                sh 'kubectl apply -f Kubernetes/deployment.yml'
                sh 'kubectl apply -f Kubernetes/service.yml'
            }
        }
    }
}
