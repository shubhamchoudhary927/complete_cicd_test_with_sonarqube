pipeline {
    agent any

    environment {
        IMAGE_NAME = "shubham6261/cicd-app"
        K8S_DEPLOYMENT = "cicd-app"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master',
                url: 'https://github.com/shubhamchoudhary927/complete_cicd_test_with_sonarqube.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh '''
                docker run --rm \
                -v $PWD:/app \
                -w /app \
                maven:3.9.6-eclipse-temurin-17 \
                mvn clean verify sonar:sonar
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:${BUILD_ID} .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker push $IMAGE_NAME:${BUILD_ID}
                docker tag $IMAGE_NAME:${BUILD_ID} $IMAGE_NAME:latest
                docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl set image deployment/cicd-app \
                cicd-app=$IMAGE_NAME:${BUILD_ID}

                kubectl rollout status deployment/cicd-app
                '''
            }
        }
    }

    post {
        success {
            echo '🚀 APPLICATION DEPLOYED SUCCESSFULLY'
        }

        failure {
            echo '❌ PIPELINE FAILED'
        }

        always {
            cleanWs()
        }
    }
}