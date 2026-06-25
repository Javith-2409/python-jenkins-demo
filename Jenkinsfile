pipeline {
    agent any

    environment {
        IMAGE_NAME = "python-project"
        DOCKER_HUB = "javithn79"
        BUILD_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "python-app"
    }

    stages {

        stage('Install & Test') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r app/requirements.txt

                export PYTHONPATH=$PWD
                pytest tests/
                '''
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {

                    sh '''
                    echo "$PASS" | docker login -u "$USER" --password-stdin

                    docker build -t $DOCKER_HUB/$IMAGE_NAME:$BUILD_TAG app/
                    docker tag $DOCKER_HUB/$IMAGE_NAME:$BUILD_TAG $DOCKER_HUB/$IMAGE_NAME:latest

                    docker push $DOCKER_HUB/$IMAGE_NAME:$BUILD_TAG
                    docker push $DOCKER_HUB/$IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                # Stop and remove old container if exists
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true

                # Run new container
                docker run -d \
                    --name $CONTAINER_NAME \
                    -p 5000:5000 \
                    $DOCKER_HUB/$IMAGE_NAME:$BUILD_TAG
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 Application deployed successfully"
            }
        }
    }

    post {
        success {
            echo "✅ Success: $DOCKER_HUB/$IMAGE_NAME:$BUILD_TAG"
        }
        failure {
            echo "❌ Failed"
        }
    }
}
