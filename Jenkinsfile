pipeline {
    agent any

    environment {
        IMAGE_NAME = "manikandan1210/python-cicd-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
        DEPLOYMENT_NAME = "python-app"
        K8S_NAMESPACE = "default"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Manikandankk12/python-cicd-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                python3 -m pip install --upgrade pip
                python3 -m pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh 'python3 -m pytest -v'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $FULL_IMAGE .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push $FULL_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl set image deployment/$DEPLOYMENT_NAME \
                python-cicd=$FULL_IMAGE \
                -n $K8S_NAMESPACE

                kubectl rollout status deployment/$DEPLOYMENT_NAME \
                -n $K8S_NAMESPACE --timeout=60s
                '''
            }
        }
    }

    post {
        failure {
            echo "❌ Deployment failed – rolling back"
            sh '''
            kubectl rollout undo deployment/$DEPLOYMENT_NAME \
            -n $K8S_NAMESPACE
            '''
        }

        success {
            echo "✅ Deployment successful with image $FULL_IMAGE"
        }
    }
}

