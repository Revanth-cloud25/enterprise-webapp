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
                ssh -o StrictHostKeyChecking=no ec2-user@10.0.12.66 << EOF
                sudo cp index.html /var/www/html/index.html
                sudo systemctl restart httpd
                EOF
                '''
            }
        }
    }
}
