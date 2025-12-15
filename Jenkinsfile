pipeline {
    agent any

    environment {
        DOCKER_USER = 'takouanaceur'
        IMAGE_NAME  = 'student-management'
        IMAGE_TAG   = 'latest'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_TOKEN')
                ]) {
                    sh '''
                        echo "$DOCKER_HUB_TOKEN" | \
                        docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f deployment.yaml -f mysql-deployment.yaml -f mysql-service.yaml -f service.yaml

                    kubectl rollout status deployment/student-management
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline terminé avec succès'
        }
        failure {
            echo '❌ Pipeline échoué'
        }
    }
}

