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
                  sh 'mvn clean test -Dspring.docker.compose.skip.in-tests=true'
            }
        }
        stage('Security Scan') {
             steps {
                 sh 'trivy fs --exit-code 0 --severity   HIGH,CRITICAL --format table . '
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
    success {
        echo 'Pipeline passed!'
        emailext(
            subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
            body: "Good news! Build ${env.BUILD_NUMBER} passed successfully.\n\nCheck it here: ${env.BUILD_URL}",
            to: 'saitejareddy.de@gmail.com'
        )
    }
    failure {
        echo 'Pipeline failed!'
        emailext(
            subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
            body: "Build ${env.BUILD_NUMBER} failed!\n\nCheck it here: ${env.BUILD_URL}",
            to: 'saitejareddy.de@gmail.com'
        )
    }
}
}
   
    