pipeline {
    agent any

    environment {
        REGISTRY = "docker.io"
        IMAGE = "joelfernandes01/wordpress"
        TAG = "latest"
        REMOTE_HOST = "docker.connect.local"
        REMOTE_USER = "docker"
    }

    triggers {
        // dispara automaticamente em push na branch main (via Webhook do GitHub)
        pollSCM('* * * * *') // opcional: fallback a cada 1 min
    }

    stages {
        stage('Checkout') {
            steps {
                sshagent(['github-key']) {
                    git branch: 'main', url: 'git@github.com:JoelFernandes01/wordpress.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        echo ">>> Construindo imagem Docker..."
                        docker build -t $REGISTRY/$IMAGE:$TAG .
                    """
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $REGISTRY/$IMAGE:$TAG
                    """
                }
            }
        }

        stage('Deploy on Remote Host') {
            steps {
                sshagent(['docker-host']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no $REMOTE_USER@$REMOTE_HOST '
                            cd ~/wordpress || git clone git@github.com:JoelFernandes01/wordpress.git ~/wordpress;
                            cd ~/wordpress && git pull;
                            docker-compose pull;
                            docker-compose up -d;
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deploy do WordPress finalizado em http://docker.connect.local:8080"
        }
        failure {
            echo "❌ Pipeline falhou! Verifique os logs."
        }
    }
}

