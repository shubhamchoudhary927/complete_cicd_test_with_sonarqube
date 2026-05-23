pipeline {
    agent any

    environment {
        IMAGE_NAME = "shubham6261/cicd-app"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/shubhamchoudhary927/complete_cicd_test_with_sonarqube.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        echo "Running SonarQube Scan..."
                        mvn clean verify sonar:sonar
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $IMAGE_NAME:${BUILD_ID} ."
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo $PASS | docker login -u $USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh "docker push $IMAGE_NAME:${BUILD_ID}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl set image deployment/cicd-app cicd-app=$IMAGE_NAME:${BUILD_ID}
                    kubectl rollout status deployment/cicd-app
                '''
            }
        }
    }

    post {
        success {
            echo "🚀 CI/CD SUCCESS - APP DEPLOYED"
        }
        failure {
            echo "❌ PIPELINE FAILED"
        }
    }
}