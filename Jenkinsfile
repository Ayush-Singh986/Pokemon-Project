pipeline {
    agent { label 'linux-agent' }

    environment {
        EC2_USER    = "ubuntu"
        EC2_HOST    = "3.110.214.168"
        DEPLOY_PATH = "/var/www/html"
        GIT_REPO    = "https://github.com/Ayush-Singh986/Pokemon-Project.git"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: env.GIT_REPO
            }
        }

        stage('Verify Application Files') {
            steps {
                sh '''
                    ls -la

                    test -f index.html || (echo "index.html missing" && exit 1)
                    test -f anime.html || (echo "anime.html missing" && exit 1)
                '''
            }
        }

        stage('Pre-deployment Health Check') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no \
                        $SSH_USER@${EC2_HOST} "
                            systemctl is-active nginx
                            ls -la ${DEPLOY_PATH}
                        "
                    '''
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        rsync -avz --delete \
                          --exclude Jenkinsfile \
                          --exclude .git \
                          --exclude README.md \
                          -e "ssh -i $SSH_KEY -o StrictHostKeyChecking=no" \
                          ./ $SSH_USER@${EC2_HOST}:${DEPLOY_PATH}/

                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no \
                        $SSH_USER@${EC2_HOST} "
                            sudo chown -R ubuntu:ubuntu ${DEPLOY_PATH}
                            sudo chmod -R 755 ${DEPLOY_PATH}
                            sudo systemctl reload nginx
                        "
                    '''
                }
            }
        }

        stage('Post-deployment Verification') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no \
                        $SSH_USER@${EC2_HOST} "
                            test -f ${DEPLOY_PATH}/index.html
                            test -f ${DEPLOY_PATH}/anime.html
                            curl -s -o /dev/null -w 'HTTP %{http_code}\n' http://localhost
                        "
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }

        success {
            echo "Deployment successful. URL: http://${EC2_HOST}"
        }

        failure {
            echo "Deployment failed. Check logs."
        }
    }
}
