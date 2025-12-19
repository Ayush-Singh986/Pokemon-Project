pipeline {
    agent { label 'linux-agent' }

    environment {
        EC2_USER = "ubuntu"  // Ubuntu EC2 default user
        EC2_HOST = "13.xxx.xxx.xxx"  // Replace with your Web Server public IP
        DEPLOY_PATH = "/var/www/html"
        GIT_REPO = "https://github.com/Ayush-Singh986/pokemon-anime-store.git"
    }

    triggers {
        // Enable GitHub webhook trigger
        githubPush()
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "🔄 Checking out code from GitHub..."
                git branch: 'main',
                    url: "${GIT_REPO}"
                
                echo "📋 Current commit information:"
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
                echo "✅ Validating application files..."
                sh '''
                echo "📁 Directory contents:"
                ls -la
                
                echo "🔍 Checking required files..."
                if [ ! -f index.html ]; then
                    echo "❌ ERROR: index.html not found!"
                    exit 1
                fi
                
                if [ ! -f anime.html ]; then
                    echo "❌ ERROR: anime.html not found!"
                    exit 1
                fi
                
                echo "✅ All required files are present"
                
                echo "📊 File sizes:"
                du -h *.html
                '''
            }
        }

        stage('Pre-deployment Health Check') {
            steps {
                echo "🏥 Performing pre-deployment health check..."
                sshagent(credentials: ['ec2-ssh']) {
                    sh '''
                    echo "🔗 Testing SSH connection to EC2..."
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "echo 'SSH connection successful'"
                    
                    echo "🌐 Checking Nginx web server status..."
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "sudo systemctl is-active nginx"
                    
                    echo "📂 Checking deployment directory..."
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "ls -la ${DEPLOY_PATH}"
                    '''
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                echo "🚀 Deploying application to AWS EC2..."
                sshagent(credentials: ['ec2-ssh']) {
                    sh '''
                    echo "📤 Starting file transfer..."
                    rsync -avz --delete \
                      --exclude Jenkinsfile \
                      --exclude .git \
                      --exclude README.md \
                      --progress \
                      ./ ${EC2_USER}@${EC2_HOST}:${DEPLOY_PATH}/
                    
                    echo "🔧 Setting proper file permissions for Ubuntu/Nginx..."
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "
                        sudo chmod -R 755 ${DEPLOY_PATH}
                        sudo chown -R ubuntu:ubuntu ${DEPLOY_PATH}
                        sudo systemctl reload nginx
                        ls -la ${DEPLOY_PATH}
                    "
                    '''
                }
            }
        }

        stage('Post-deployment Verification') {
            steps {
                echo "🔍 Verifying deployment..."
                sshagent(credentials: ['ec2-ssh']) {
                    sh '''
                    echo "📋 Checking deployed files..."
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "
                        echo 'Files in deployment directory:'
                        ls -la ${DEPLOY_PATH}
                        
                        echo 'Checking if index.html exists:'
                        test -f ${DEPLOY_PATH}/index.html && echo '✅ index.html deployed successfully' || echo '❌ index.html missing'
                        
                        echo 'Checking if anime.html exists:'
                        test -f ${DEPLOY_PATH}/anime.html && echo '✅ anime.html deployed successfully' || echo '❌ anime.html missing'
                    "
                    
                    echo "🌐 Testing web server response..."
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "
                        curl -s -o /dev/null -w 'HTTP Status: %{http_code}' http://localhost/ || echo 'Web server test failed'
                    "
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning up workspace..."
            cleanWs()
        }
        success {
            echo """
            🎉 DEPLOYMENT SUCCESSFUL! 🎉
            
            ✅ Application deployed successfully to AWS EC2
            🌐 Website URL: http://${EC2_HOST}
            📁 Deployment Path: ${DEPLOY_PATH}
            🕒 Deployment Time: ${new Date()}
            
            🔗 Access your Pokemon & Anime Store at: http://${EC2_HOST}
            """
        }
        failure {
            echo """
            ❌ DEPLOYMENT FAILED! ❌
            
            💥 The deployment pipeline has failed
            🔍 Check the console output above for error details
            🛠️  Common issues:
               - SSH connection problems
               - File permission issues
               - Web server not running
               - Network connectivity issues
            
            📞 Contact the DevOps team for assistance
            """
        }
        unstable {
            echo "⚠️ Deployment completed with warnings. Please review the build logs."
        }
    }
}
