pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('List Files') {
            steps {
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                scp -r -o StrictHostKeyChecking=no app ec2-user@10.0.11.85:/tmp/
                scp -o StrictHostKeyChecking=no scripts/deploy.sh ec2-user@10.0.11.85:/tmp/

                ssh -o StrictHostKeyChecking=no ec2-user@10.0.11.85 "
                    chmod +x /tmp/deploy.sh
                    sudo bash /tmp/deploy.sh
                "
                '''
            }
        }
    }
}
