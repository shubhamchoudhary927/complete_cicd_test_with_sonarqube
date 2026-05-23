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
                -v "$WORKSPACE:/app" \
                -w /app \
                maven:3.9.6-eclipse-temurin-17 \
                mvn clean verify sonar:sonar
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $IMAGE_NAME:${BUILD_ID} ."
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
                    echo "$PASS" | docker login -u "$USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh "docker push $IMAGE_NAME:${BUILD_ID}"
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh """
                kubectl set image deployment/${K8S_DEPLOYMENT} \
                ${K8S_DEPLOYMENT}=$IMAGE_NAME:${BUILD_ID}

                kubectl rollout status deployment/${K8S_DEPLOYMENT}
                """
            }
        }
    }

    post {
        success {
            echo '🚀 PIPELINE SUCCESS'
        }

        failure {
            echo '❌ PIPELINE FAILED'
        }

        always {
            cleanWs()
        }
    }
}