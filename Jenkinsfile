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
<<<<<<< HEAD
                scp -r -o StrictHostKeyChecking=no app ec2-user@10.0.11.85:/tmp/
                scp -o StrictHostKeyChecking=no scripts/deploy.sh ec2-user@10.0.11.85:/tmp/

                ssh -o StrictHostKeyChecking=no ec2-user@10.0.11.85 "
                    chmod +x /tmp/deploy.sh
                    sudo bash /tmp/deploy.sh
=======
                APP_IP=$(aws ec2 describe-instances \
                --filters "Name=tag:Name,Values=enterprise-webapp-dev-app-server" \
                          "Name=instance-state-name,Values=running" \
                --query "Reservations[0].Instances[0].PrivateIpAddress" \
                --output text)

                echo "Deploying to $APP_IP"

                scp -r -o StrictHostKeyChecking=no app ec2-user@$APP_IP:/tmp/

                ssh -o StrictHostKeyChecking=no ec2-user@$APP_IP "
                sudo rm -rf /var/www/html/*
                sudo cp -r /tmp/app/* /var/www/html/
                sudo systemctl restart httpd
>>>>>>> 1726efa (Implement deployment scripts changed)
                "
                '''
            }
        }
<<<<<<< HEAD
=======

>>>>>>> 1726efa (Implement deployment scripts changed)
    }
}
