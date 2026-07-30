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
                scp -o StrictHostKeyChecking=no index.html ec2-user@10.0.12.66:/tmp/index.html

                ssh -o StrictHostKeyChecking=no ec2-user@10.0.12.66 << EOF
                sudo mv /tmp/index.html /var/www/html/index.html
                sudo systemctl restart httpd
                EOF
                '''
            }
        }
    }
}