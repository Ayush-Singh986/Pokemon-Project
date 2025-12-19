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
                echo "Checking out code from GitHub"

                git branch: 'main',
                    url: env.GIT_REPO

                sh '''
                    echo "Commit ID: $(git rev-parse HEAD)"
                    echo "Commit Message: $(git log -1 --pretty=%B)"
                    echo "Author: $(git log -1 --pretty=%an)"
                    echo "Date: $(git log -1 --pretty=%ad)"
                '''
            }
        }

        stage('Verify Application Files') {
            steps {
                echo "Verifying application files"
                sh '''
                    ls -la

                    if [ ! -f index.html ]; then
                        echo "ERROR: index.html not found"
                        exit 1
                    fi

                    if [ ! -f anime.html ]; then
                        echo "ERROR: anime.html not found"
                        exit 1
                    fi

                    du -h *.html
                '''
            }
        }

        stage('Pre-deployment Health Check') {
            steps {
                echo "Running pre-deployment health checks"
                sshagent(credentials: ['ec2-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "
                            systemctl is-active nginx
                            ls -la ${DEPLOY_PATH}
                        "
                    '''
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                echo "Deploying application to EC2"
                sshagent(credentials: ['ec2-ssh']) {
                    sh '''
                        rsync -avz --delete \
                          --exclude Jenkinsfile \
                          --exclude .git \
                          --exclude README.md \
                          ./ ${EC2_USER}@${EC2_HOST}:${DEPLOY_PATH}/

                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "
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
                echo "Verifying deployment"
                sshagent(credentials: ['ec2-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "
                            test -f ${DEPLOY_PATH}/index.html
                            test -f ${DEPLOY_PATH}/anime.html
                            curl -s -o /dev/null -w 'HTTP Status: %{http_code}\n' http://localhost
                        "
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace"
            cleanWs()
        }

        success {
            echo """
DEPLOYMENT SUCCESSFUL

URL: http://${EC2_HOST}
Deployment Path: ${DEPLOY_PATH}
Time: ${new Date()}
"""
        }

        failure {
            echo """
DEPLOYMENT FAILED

Check Jenkins console output for errors.
Possible causes:
- SSH credential issue
- Nginx not running
- Permission problems
- Network or security group issues
"""
        }
    }
}
