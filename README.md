# Pokemon & Anime Store - CI/CD Deployment Project

## Introduction

This project demonstrates a complete CI/CD pipeline for deploying a static web application (Pokemon & Anime Store) to AWS EC2 using Jenkins. The project showcases automated deployment practices, infrastructure management, and continuous delivery workflows.

## Project Overview

The project includes:
* A responsive Pokemon merchandise store (index.html)
* An anime characters collectibles page (anime.html)
* Automated CI/CD pipeline using Jenkins with GitHub webhooks
* Automatic deployment triggered by GitHub commits
* Deployment to AWS EC2 instances
* Infrastructure automation with rsync
* Complete GitOps workflow

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Project Structure](#project-structure)
3. [Local Development Setup](#local-development-setup)
4. [AWS EC2 Setup](#aws-ec2-setup)
5. [Jenkins Configuration](#jenkins-configuration)
6. [GitHub Webhook Setup](#github-webhook-setup)
7. [Deployment Process](#deployment-process)
8. [Automatic CI/CD Workflow](#automatic-cicd-workflow)
9. [Testing the Application](#testing-the-application)
10. [Troubleshooting](#troubleshooting)
11. [Project Cleanup](#project-cleanup)

---

## Prerequisites

Before starting, ensure you have:

* **AWS Account** with EC2 access
* **Jenkins Server** (installed and running)
* **Git** installed locally
* **SSH Key Pair** for EC2 access
* **Web Browser** for testing
* Basic knowledge of HTML, Jenkins, and AWS

---

## Project Structure

```
pokemon-anime-store/
├── README.md           # Project documentation
├── index.html          # Pokemon store homepage
├── anime.html          # Anime characters page
└── Jenkinsfile         # Jenkins pipeline configuration
```

---

## Local Development Setup

### 1. Clone the Repository

```bash
git clone https://github.com/Ayush-Singh986/pokemon-anime-store.git
cd pokemon-anime-store
```

### 2. View Files Locally

You can open the HTML files directly in your browser:

```bash
# On Linux/Mac
open index.html

# On Windows
start index.html
```

Or use a simple HTTP server:

```bash
# Using Python 3
python -m http.server 8000

# Using Python 2
python -m SimpleHTTPServer 8000

# Using Node.js (if http-server is installed)
npx http-server -p 8000
```

Then navigate to `http://localhost:8000` in your browser.

### 3. Verify Application Structure

Check that all files are present:

```bash
ls -la
```

You should see:
* `index.html` - Main Pokemon store page
* `anime.html` - Anime characters page
* `Jenkinsfile` - CI/CD pipeline configuration

---

## AWS EC2 Setup

### 1. Launch an EC2 Instance

1. **Log in to AWS Console**
2. **Navigate to EC2 Dashboard**
3. **Click "Launch Instance"**

4. **Configure Instance:**
   * **Name:** `pokemon-store-server`
   * **AMI:** Amazon Linux 2 or Ubuntu 20.04
   * **Instance Type:** t2.micro (free tier eligible)
   * **Key Pair:** Create or select existing key pair
   * **Security Group:** Allow HTTP (80), HTTPS (443), and SSH (22)

5. **Launch the Instance**

### 2. Install Web Server on EC2

Connect to your EC2 instance:

```bash
ssh -i your-key.pem ec2-user@your-ec2-public-ip
```

#### For Amazon Linux 2:

```bash
# Update system packages
sudo yum update -y

# Install Apache web server
sudo yum install httpd -y

# Start Apache service
sudo systemctl start httpd

# Enable Apache to start on boot
sudo systemctl enable httpd

# Verify Apache is running
sudo systemctl status httpd
```

#### For Ubuntu:

```bash
# Update system packages
sudo apt update -y

# Install Nginx web server
sudo apt install nginx -y

# Start Nginx service
sudo systemctl start nginx

# Enable Nginx to start on boot
sudo systemctl enable nginx

# Verify Nginx is running
sudo systemctl status nginx
```

### 3. Configure Web Server Directory

```bash
# Set proper permissions for deployment directory
sudo chmod 755 /var/www/html

# Change ownership (optional, for easier deployment)
sudo chown -R ec2-user:ec2-user /var/www/html
```

### 4. Test Web Server

Open your browser and navigate to:
```
http://your-ec2-public-ip
```

You should see the default Apache or Nginx welcome page.

---

## Jenkins Configuration

### 1. Install Jenkins

#### On Ubuntu:

```bash
# Add Jenkins repository
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

# Install Jenkins
sudo apt update
sudo apt install jenkins -y

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

#### On Amazon Linux 2:

```bash
# Add Jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

# Install Java (required for Jenkins)
sudo yum install java-11-openjdk -y

# Install Jenkins
sudo yum install jenkins -y

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 2. Access Jenkins Web Interface

Navigate to:
```
http://your-jenkins-server-ip:8080
```

Complete the initial setup wizard.

### 3. Install Required Jenkins Plugins

Go to **Manage Jenkins** → **Manage Plugins** → **Available**

Install these plugins:
* **Git Plugin** (for repository integration)
* **SSH Agent Plugin** (for SSH key management)
* **Pipeline Plugin** (for Jenkinsfile support)
* **GitHub Integration Plugin** (for webhook support)
* **Generic Webhook Trigger Plugin** (for automatic builds)

### 4. Configure Jenkins Agent (Optional)

If using a separate Linux agent:

1. Go to **Manage Jenkins** → **Manage Nodes and Clouds**
2. Click **New Node**
3. Name: `linux-agent`
4. Type: **Permanent Agent**
5. Configure SSH connection details

### 5. Add EC2 SSH Credentials to Jenkins

1. Go to **Manage Jenkins** → **Manage Credentials**
2. Click **(global)** → **Add Credentials**
3. **Kind:** SSH Username with private key
4. **ID:** `ec2-ssh`
5. **Username:** `ec2-user` (or `ubuntu` for Ubuntu)
6. **Private Key:** Paste your EC2 private key content
7. Click **OK**

---

## GitHub Webhook Setup

### 1. Configure Jenkins for GitHub Webhooks

#### Enable GitHub Webhook Support in Jenkins

1. Go to **Manage Jenkins** → **Configure System**
2. Scroll to **GitHub** section
3. Click **Add GitHub Server**
4. **Name:** `GitHub`
5. **API URL:** `https://api.github.com` (default)
6. **Credentials:** Add GitHub personal access token (optional for public repos)

#### Configure Jenkins Security for Webhooks

1. Go to **Manage Jenkins** → **Configure Global Security**
2. Under **CSRF Protection**, uncheck **Enable proxy compatibility** (if needed)
3. Or add GitHub webhook URLs to **Proxy Compatibility** list

### 2. Make Jenkins Accessible to GitHub

#### Option A: Public Jenkins Server

If Jenkins is on a public server:
```bash
# Ensure Jenkins is accessible on port 8080
sudo ufw allow 8080  # Ubuntu
sudo firewall-cmd --permanent --add-port=8080/tcp  # CentOS/RHEL
```

#### Option B: Use ngrok for Local Jenkins (Development)

If Jenkins is running locally:
```bash
# Install ngrok
wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
unzip ngrok-stable-linux-amd64.zip
sudo mv ngrok /usr/local/bin/

# Expose Jenkins to internet
ngrok http 8080
```

Copy the `https://xxxxx.ngrok.io` URL for webhook configuration.

### 3. Configure GitHub Repository Webhook

1. **Go to your GitHub repository**
2. **Click Settings** → **Webhooks**
3. **Click "Add webhook"**

4. **Configure Webhook:**
   * **Payload URL:** `http://your-jenkins-server:8080/github-webhook/`
   * **Content type:** `application/json`
   * **Secret:** (leave empty or add secret for security)
   * **Events:** Select "Just the push event"
   * **Active:** ✅ Checked

5. **Click "Add webhook"**

### 4. Test Webhook Connection

1. GitHub will automatically send a test payload
2. Check the webhook delivery in GitHub (green checkmark = success)
3. If failed, verify Jenkins URL is accessible from internet

---

## Deployment Process

### 1. Create Jenkins Pipeline Job

1. **Click "New Item"** in Jenkins
2. **Enter name:** `pokemon-store-deployment`
3. **Select:** Pipeline
4. **Click OK**

### 2. Configure Pipeline

In the pipeline configuration:

1. **Pipeline Definition:** Pipeline script from SCM
2. **SCM:** Git
3. **Repository URL:** `https://github.com/Ayush-Singh986/pokemon-anime-store.git`
4. **Branch:** `main`
5. **Script Path:** `Jenkinsfile`

#### Enable Automatic Builds

6. **Build Triggers:** Check ✅ **GitHub hook trigger for GITScm polling**
7. **Additional Behaviours:** Add **"Polling ignores commits with certain messages"** (optional)

This configuration allows Jenkins to automatically trigger builds when GitHub sends webhook notifications.

### 3. Update Jenkinsfile

Before running, update the `Jenkinsfile` with your EC2 details:

```groovy
environment {
    EC2_USER = "ec2-user"              # Change to "ubuntu" if using Ubuntu
    EC2_HOST = "your-ec2-public-ip"    # Replace with your actual EC2 IP
    DEPLOY_PATH = "/var/www/html"
}
```

### 4. Run the Pipeline

1. Click **Build Now**
2. Monitor the build progress in **Console Output**

The pipeline will:
* **Stage 1:** Checkout code from GitHub
* **Stage 2:** Verify application files exist
* **Stage 3:** Deploy files to EC2 using rsync

---

## Automatic CI/CD Workflow

### How the Automatic Deployment Works

1. **Developer pushes code** to GitHub repository
2. **GitHub webhook** automatically notifies Jenkins
3. **Jenkins receives webhook** and triggers the pipeline
4. **Pipeline executes** all stages automatically:
   - Checks out latest code from GitHub
   - Validates application files
   - Deploys to AWS EC2 server
5. **Application is live** with latest changes

### Workflow Diagram

```
Developer → Git Push → GitHub → Webhook → Jenkins → Deploy → AWS EC2
    ↓           ↓         ↓         ↓         ↓         ↓        ↓
  Code       Commit   Triggers  Receives  Executes  Updates  Live App
 Changes    to Repo   Webhook   Payload   Pipeline   Files   Available
```

### Testing Automatic Deployment

#### 1. Make a Code Change

Edit any file in your local repository:

```bash
# Clone the repository (if not already done)
git clone https://github.com/Ayush-Singh986/pokemon-anime-store.git
cd pokemon-anime-store

# Make a simple change to index.html
sed -i 's/Welcome to the Pokémon Store/Welcome to the Amazing Pokémon Store/g' index.html

# Or edit manually with your preferred editor
nano index.html
```

#### 2. Commit and Push Changes

```bash
# Add changes to git
git add .

# Commit with a descriptive message
git commit -m "Update store welcome message"

# Push to GitHub (this will trigger the webhook)
git push origin main
```

#### 3. Monitor Automatic Build

1. **Check GitHub webhook delivery:**
   - Go to repository **Settings** → **Webhooks**
   - Click on your webhook
   - Check **Recent Deliveries** tab
   - Should show successful delivery (green checkmark)

2. **Monitor Jenkins build:**
   - Go to Jenkins dashboard
   - You should see the pipeline automatically start building
   - Click on the build number to see progress
   - Monitor **Console Output** for deployment status

#### 4. Verify Deployment

```bash
# Check the live application
curl http://your-ec2-public-ip

# Or open in browser
http://your-ec2-public-ip
```

You should see your changes reflected immediately after the pipeline completes.

### Advanced Webhook Configuration

#### Webhook Security (Recommended for Production)

1. **Add webhook secret in GitHub:**
   ```
   Settings → Webhooks → Edit webhook → Secret: your-secret-key
   ```

2. **Configure Jenkins to validate webhook signature:**
   - Install **GitHub Integration Plugin**
   - Add webhook secret in Jenkins credentials
   - Configure pipeline to validate GitHub signatures

#### Branch-Specific Deployments

Update your Jenkinsfile to deploy different branches to different environments:

```groovy
pipeline {
    agent { label 'linux-agent' }
    
    environment {
        EC2_USER = "ec2-user"
        DEPLOY_PATH = "/var/www/html"
    }
    
    stages {
        stage('Determine Environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.EC2_HOST = "production-server-ip"
                        env.ENVIRONMENT = "production"
                    } else if (env.BRANCH_NAME == 'staging') {
                        env.EC2_HOST = "staging-server-ip"
                        env.ENVIRONMENT = "staging"
                    } else {
                        env.EC2_HOST = "dev-server-ip"
                        env.ENVIRONMENT = "development"
                    }
                }
            }
        }
        
        // ... rest of your stages
    }
}
```

---

## Testing the Application

### 1. Access the Application

Open your browser and navigate to:
```
http://your-ec2-public-ip
```

You should see the Pokemon Store homepage.

### 2. Test Navigation

* Click on **"Go to Famous Anime Store"** button
* Verify the anime.html page loads correctly
* Click **"Back to Home"** to return to index.html

### 3. Test Responsive Design

* Resize your browser window
* Test on mobile devices
* Verify all images and buttons work correctly

---

## Troubleshooting

### Issue: Jenkins Cannot Connect to EC2

**Solution:**
```bash
# Verify SSH key permissions
chmod 400 your-key.pem

# Test SSH connection manually
ssh -i your-key.pem ec2-user@your-ec2-ip

# Check EC2 security group allows SSH from Jenkins server IP
```

### Issue: Files Not Deploying

**Solution:**
```bash
# Check deployment directory permissions on EC2
ls -la /var/www/html

# Ensure web server is running
sudo systemctl status httpd  # or nginx

# Check Jenkins console output for errors
```

### Issue: Website Shows 403 Forbidden

**Solution:**
```bash
# Fix file permissions on EC2
sudo chmod -R 755 /var/www/html
sudo chown -R ec2-user:ec2-user /var/www/html

# Restart web server
sudo systemctl restart httpd  # or nginx
```

### Issue: Images Not Loading

**Solution:**
* Check internet connectivity on EC2
* Verify external image URLs are accessible
* Check browser console for errors (F12)

### Issue: Webhook Not Triggering Jenkins

**Solution:**
```bash
# Check webhook delivery in GitHub
# Repository → Settings → Webhooks → Recent Deliveries

# Verify Jenkins webhook URL is accessible
curl -X POST http://your-jenkins-server:8080/github-webhook/

# Check Jenkins logs
sudo tail -f /var/log/jenkins/jenkins.log

# Ensure GitHub Integration Plugin is installed and configured
```

### Issue: Jenkins Build Not Starting Automatically

**Solution:**
1. Verify **"GitHub hook trigger for GITScm polling"** is enabled in job configuration
2. Check Jenkins system log for webhook reception
3. Ensure Jenkins server is accessible from internet
4. Verify webhook payload URL format: `http://jenkins-server:8080/github-webhook/`

### Issue: Multiple Builds Triggering

**Solution:**
```bash
# Add build trigger conditions in Jenkinsfile
when {
    branch 'main'  // Only build main branch
}

# Or configure webhook to ignore certain commit messages
# In Jenkins job: Build Triggers → Advanced → Polling ignores commits with certain messages
```

---

## Project Cleanup

### 1. Stop Jenkins Job

In Jenkins:
* Disable the pipeline job
* Or delete the job entirely

### 2. Terminate EC2 Instance

In AWS Console:
1. Go to EC2 Dashboard
2. Select your instance
3. **Instance State** → **Terminate Instance**

### 3. Clean Up Local Files (Optional)

```bash
cd ..
rm -rf pokemon-anime-store
```

---

## How to Write a README

Based on this example, here are key principles for writing effective README files:

### 1. Structure

* **Clear Title** - Project name and brief description
* **Table of Contents** - For easy navigation
* **Sections** - Logical grouping of information
* **Code Blocks** - Formatted commands and code snippets

### 2. Essential Sections

* **Introduction** - What the project does
* **Prerequisites** - What users need before starting
* **Installation** - Step-by-step setup instructions
* **Usage** - How to use the project
* **Configuration** - Environment-specific settings
* **Troubleshooting** - Common issues and solutions
* **Cleanup** - How to remove/uninstall

### 3. Best Practices

* Use **clear headings** and subheadings
* Include **code examples** with syntax highlighting
* Add **screenshots** or diagrams where helpful
* Provide **actual commands** users can copy-paste
* Explain **why** not just **how**
* Keep it **up-to-date** with code changes
* Use **consistent formatting** throughout

### 4. Markdown Tips

```markdown
# H1 Heading
## H2 Heading
### H3 Heading

**Bold text**
*Italic text*

`inline code`

```bash
# Code block with syntax highlighting
command here
```

* Bullet point
1. Numbered list

[Link text](URL)
![Image alt text](image-url)
```

---

## Conclusion

You've successfully set up a complete automated CI/CD pipeline for deploying a static web application to AWS EC2 using Jenkins with GitHub webhooks. This project demonstrates:

* **Version Control** with Git and GitHub
* **Continuous Integration** with Jenkins
* **Automated Deployment** triggered by code changes
* **Webhook Integration** for real-time deployment
* **Cloud Infrastructure** with AWS EC2
* **Web Server Configuration** with Apache/Nginx
* **GitOps Workflow** with automatic deployment pipeline

You can now apply these concepts to deploy more complex applications, add automated testing, implement blue-green deployments, or integrate with other cloud services.

---

## Additional Resources

* [Jenkins Documentation](https://www.jenkins.io/doc/)
* [AWS EC2 User Guide](https://docs.aws.amazon.com/ec2/)
* [Git Documentation](https://git-scm.com/doc)
* [Markdown Guide](https://www.markdownguide.org/)

---

**Project Repository:** https://github.com/Ayush-Singh986/pokemon-anime-store

**License:** MIT

**Author:** Ayush Singh
