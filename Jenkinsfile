pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    triggers {
        githubPush()
    }

    environment {
        SONAR_PROJECT_KEY = "java-backend-app"
        SONAR_PROJECT_NAME = "java-backend-app"
        NEXUS_URL = "http://192.168.0.50:9090/repository/maven_releases/"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/PoojaBusa09/java-backend-app.git'
            }
        }

        stage('Clean Build') {
            steps {
                sh 'mvn clean'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                        sh """
                            mvn sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName=${SONAR_PROJECT_NAME}
                        """
                    }
                }
            }
        

        stage('Nexus Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-cred', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                        mvn deploy -DskipTests \
                        -DaltDeploymentRepository=nexus::default::${NEXUS_URL} \
                        -Dnexus.username=$NEXUS_USER \
                        -Dnexus.password=$NEXUS_PASS
                    """
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t java-backend-app .'
            }
        }

        stage('Run Container') {
            steps {
                sh """
                    docker rm -f backend-app || true
                    docker run -d -p 8081:8081 --name backend-app java-backend-app
                """
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                sh """
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    kubectl rollout status deployment/java-backend-app-deployment
                """
            }
        }
    }

    post {
        success {
            echo "✔ Pipeline SUCCESS - Deployment completed"
        }
        failure {
            echo "❌ Pipeline FAILED - Check logs"
        }
    }
}
