pipeline {
    agent { label "agent1" }

    environment {
        IMAGE_NAME = "shivanshks06/notes-app"
        TAG = "${BUILD_NUMBER}"  
    }

    stages {

        stage('Clean Workspace') {
            steps {
                echo "Cleaning workspace"
                deleteDir()
            }
        }

        stage('Cleanup Docker') {
            steps {
                echo "Cleaning unused Docker resources"

                // safer cleanup
                sh "docker container prune -f"
                sh "docker image prune -f"
            }
        }

        stage('Code') {
            steps {
                echo "cloning code"
                git url: "https://github.com/shivanshks06/django-notes-app.git/", branch: "main"
            }
        }

        stage('Build') {
            steps {
                echo "building code"

                sh "docker build -t $IMAGE_NAME:$TAG ."
                sh "docker tag $IMAGE_NAME:$TAG $IMAGE_NAME:latest"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Pushing image to Docker Hub"

                withCredentials([usernamePassword(
                    credentialsId: "dockerHub",
                    usernameVariable: "DOCKER_USER",
                    passwordVariable: "DOCKER_PASS"
                )]) {

                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:$TAG
                    docker push $IMAGE_NAME:latest
                    docker logout
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "deploying code"
                sh "docker pull $IMAGE_NAME:latest"
                sh "docker compose down || true"
                sh "docker compose up -d"
            }
        }
    }
}
