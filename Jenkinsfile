pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube'  // Name configured in Jenkins > Global Tool Configuration
    }

    stages {

        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'mvn clean install'
            }
        }

        stage('Unit and Integration Tests') {
            steps {
                echo 'Running unit and integration tests...'
                sh 'mvn test'
            }
        }

        stage('Code Analysis') {
            steps {
                echo 'Running code quality analysis using SonarQube...'
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Security Scan') {
            steps {
                echo 'Performing security scan with OWASP Dependency-Check...'
                sh 'mvn org.owasp:dependency-check-maven:check'
            }
        }

        stage('Deploy to Staging') {
            steps {
                echo 'Deploying to staging server...'
                sh '''
                    scp target/app.jar ec2-user@staging-server:/opt/app/
                    ssh ec2-user@staging-server 'pkill -f app.jar || true && nohup java -jar /opt/app/app.jar > /dev/null 2>&1 &'
                '''
            }
        }

        stage('Integration Tests on Staging') {
            steps {
                echo 'Running integration tests on staging...'
                sh 'newman run tests/postman_collection.json'
            }
        }

        stage('Deploy to Production') {
            steps {
                input message: 'Approve deployment to production?', ok: 'Deploy'
                echo 'Deploying to production server...'
                sh '''
                    scp target/app.jar ec2-user@prod-server:/opt/app/
                    ssh ec2-user@prod-server 'pkill -f app.jar || true && nohup java -jar /opt/app/app.jar > /dev/null 2>&1 &'
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
