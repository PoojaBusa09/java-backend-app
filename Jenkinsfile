pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/PoojaBusa09/java-backend-app.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
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
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=java-backend-app \
                        -Dsonar.projectName=java-backend-app
                    '''
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
                sh 'docker run -d -p 8081:8081 --name backend-app java-backend-app'
            }
        }
    }
}
