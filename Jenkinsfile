pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    environment {
        SONAR_PROJECT_KEY = "java-backend-app"
        SONAR_PROJECT_NAME = "java-backend-app"
        SONAR_HOST_URL = "http://192.168.0.50:9000"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/PoojaBusa09/java-backend-app.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {

                        // IMPORTANT: using single quotes to avoid token issues
                        sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                        -Dsonar.projectName=$SONAR_PROJECT_NAME \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()

                        if (qg.status != 'OK') {
                            error "❌ Pipeline aborted due to Quality Gate failure: ${qg.status}"
                        } else {
                            echo "✔ Quality Gate PASSED"
                        }
                    }
                }
            }
        }

        stage('Nexus Deploy') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-cred',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh '''
                    mvn deploy -DskipTests \
                    -Dnexus.username=$NEXUS_USER \
                    -Dnexus.password=$NEXUS_PASS
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t java-backend-app:latest .'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker rm -f backend-app || true
                docker run -d -p 8081:8081 --name backend-app java-backend-app:latest
                '''
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                sh '''
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                kubectl rollout status deployment/java-backend-app-deployment
                '''
            }
        }
    }

    post {
        success {
            echo "✔ PIPELINE SUCCESS - Deployment Completed"
        }
        failure {
            echo "❌ PIPELINE FAILED - Check Sonar/Auth logs"
        }
    }
}
