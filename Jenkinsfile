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
                scp -o StrictHostKeyChecking=no index.html ec2-user@10.0.11.85:/tmp/index.html

                ssh -o StrictHostKeyChecking=no ec2-user@10.0.11.85 << EOF
                sudo mv /tmp/index.html /var/www/html/index.html
                sudo systemctl restart httpd
                EOF
                '''
            }
        }
    }
}