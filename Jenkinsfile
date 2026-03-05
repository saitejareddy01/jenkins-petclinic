pipeline {
    agent { label 'maven-agent' }

    environment {
        APP_SERVER = '172.31.0.182'
        APP_DIR = '/home/ubuntu/app'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                  sh 'mvn clean test -Dexclude=**/*PostgresIntegrationTests.java,**/*MySqlIntegrationTests.java'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy to App Server') {
            when {
                branch 'main'
            }
            steps {
                sshagent(['app-server-ssh']) {
                    sh '''
                        scp -i ~/.ssh/jenkins_rsa -o StrictHostKeyChecking=no target/*.jar ubuntu@172.31.0.182:/home/ubuntu/app/app.jar
                        ssh -i ~/.ssh/jenkins_rsa -o StrictHostKeyChecking=no ubuntu@172.31.0.182 "pkill -f app.jar || true && nohup java -jar /home/ubuntu/app/app.jar > /home/ubuntu/app/app.log 2>&1 &"
                    '''
                }
            }
        }
    }

    post {
        success { echo 'Pipeline passed!' }
        failure { echo 'Pipeline failed!' }
    }
}