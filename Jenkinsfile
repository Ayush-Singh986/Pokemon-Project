pipeline {
    agent { label 'linux-agent' }

    environment {
        EC2_USER = "ec2-user"
        EC2_HOST = "13.xxx.xxx.xxx" 
        DEPLOY_PATH = "/var/www/html"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Ayush-Singh986/pokemon-anime-store.git'
            }
        }

        stage('Verify Application Files') {
            steps {
                sh '''
                echo "Validating files..."
                ls -l
                test -f index.html
                test -f anime.html
                '''
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                sshagent(credentials: ['ec2-ssh']) {
                    sh '''
                    rsync -avz --delete \
                      --exclude Jenkinsfile \
                      ./ ${EC2_USER}@${EC2_HOST}:${DEPLOY_PATH}/
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Application deployed successfully to /var/www/html"
        }
        failure {
            echo "Deployment failed"
        }
    }
}
