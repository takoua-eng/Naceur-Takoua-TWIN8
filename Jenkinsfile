pipeline {
    agent any

    environment {
        DOCKER_USER = 'takouanaceur'
        APP_IMAGE   = 'takouanaceur/student-management:latest'
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $APP_IMAGE .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_TOKEN')]) {
                    sh 'echo $DOCKER_HUB_TOKEN | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $APP_IMAGE'
            }
        }

        stage('Deploy MySQL to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh '''
                      kubectl apply -f mysql-deployment.yaml
                      kubectl apply -f mysql-service.yaml
                    '''
                }
            }
        }

        stage('Deploy Spring Boot to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh '''
                      kubectl apply -f deployment.yaml
                      kubectl apply -f service.yaml
                      kubectl rollout status deployment/student-management
                    '''
                }
            }
        }

        stage('K8s Smoke Test') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh '''
                      kubectl get pods
                      kubectl get svc
                    '''
                }
            }
        }
    }
}

