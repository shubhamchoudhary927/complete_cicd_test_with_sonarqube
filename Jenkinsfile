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
                withSonarQubeEnv('sonar') {
                    sh '''
                        mvn clean verify sonar:sonar
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
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
                sh """
                    kubectl set image deployment/${K8S_DEPLOYMENT} ${K8S_DEPLOYMENT}=$IMAGE_NAME:${BUILD_ID}
                    kubectl rollout status deployment/${K8S_DEPLOYMENT} --timeout=120s
                """
            }
        }
    }

    post {
        success {
            echo "🚀 SUCCESS: CI/CD pipeline completed successfully"
        }

        failure {
            echo "❌ FAILED: Check logs"
        }

        always {
            cleanWs()
        }
    }
}