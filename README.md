# Pokemon & Anime Store - CI/CD Deployment Project

## Introduction

This project demonstrates a complete CI/CD pipeline for deploying a static web application (Pokemon & Anime Store) running entirely on AWS EC2 Ubuntu servers. The project showcases automated deployment practices, infrastructure management, and continuous delivery workflows using Jenkins hosted on Ubuntu EC2.

## Project Overview

The project includes:
* A responsive Pokemon merchandise store (index.html)
* An anime characters collectibles page (anime.html)
* Jenkins CI/CD server running on AWS EC2 Ubuntu
* Web application hosted on AWS EC2 Ubuntu with Nginx
* Automated CI/CD pipeline using Jenkins with GitHub webhooks
* Automatic deployment triggered by GitHub commits
* Infrastructure automation with rsync between Ubuntu servers
* Complete GitOps workflow on AWS cloud infrastructure

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

* **AWS Account** with EC2 access and permissions
* **Two Ubuntu EC2 instances** (one for Jenkins, one for web server)
* **SSH Key Pair** for EC2 access
* **Security Groups** configured for HTTP, HTTPS, and SSH access
* **Git** installed locally for development
* **Web Browser** for testing
* Basic knowledge of HTML, Jenkins, Ubuntu, and AWS

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

## AWS EC2 Ubuntu Setup

### 1. Launch Ubuntu EC2 Instances

You'll need **two Ubuntu EC2 instances**:
1. **Jenkins Server** - For CI/CD pipeline
2. **Web Server** - For hosting the application

#### Launch Jenkins Server Instance

1. **Log in to AWS Console**
2. **Navigate to EC2 Dashboard**
3. **Click "Launch Instance"**

4. **Configure Jenkins Server:**
   * **Name:** `jenkins-server-ubuntu`
   * **AMI:** Ubuntu Server 22.04 LTS (Free tier eligible)
   * **Instance Type:** t2.small (recommended for Jenkins) or t2.micro
   * **Key Pair:** Create or select existing key pair
   * **Security Group:** Allow SSH (22), HTTP (8080 for Jenkins)

#### Launch Web Server Instance

5. **Launch second instance for web server:**
   * **Name:** `pokemon-store-web-server`
   * **AMI:** Ubuntu Server 22.04 LTS (Free tier eligible)
   * **Instance Type:** t2.micro (free tier eligible)
   * **Key Pair:** Same key pair as Jenkins server
   * **Security Group:** Allow SSH (22), HTTP (80), HTTPS (443)

#### Configure Security Groups

**Jenkins Server Security Group:**
```
Port 22 (SSH) - Your IP address
Port 8080 (Jenkins) - 0.0.0.0/0 (or your IP for security)
Port 8080 (Jenkins) - GitHub webhook IPs (for automatic triggers)
```

**Web Server Security Group:**
```
Port 22 (SSH) - Jenkins server IP (for deployment)
Port 80 (HTTP) - 0.0.0.0/0 (public access)
Port 443 (HTTPS) - 0.0.0.0/0 (public access)
```

### 2. Setup Web Server (Ubuntu EC2)

Connect to your **Web Server** Ubuntu instance:

```bash
ssh -i your-key.pem ubuntu@your-web-server-public-ip
```

#### Install and Configure Nginx

```bash
# Update system packages
sudo apt update -y
sudo apt upgrade -y

# Install Nginx web server
sudo apt install nginx -y

# Start Nginx service
sudo systemctl start nginx

# Enable Nginx to start on boot
sudo systemctl enable nginx

# Verify Nginx is running
sudo systemctl status nginx

# Check if Nginx is serving content
curl http://localhost
```

#### Configure Nginx for Application

```bash
# Set proper permissions for deployment directory
sudo chmod 755 /var/www/html

# Change ownership for easier deployment
sudo chown -R ubuntu:ubuntu /var/www/html

# Remove default Nginx page
sudo rm /var/www/html/index.nginx-debian.html

# Create a simple test page
echo "<h1>Pokemon Store Server Ready!</h1>" | sudo tee /var/www/html/index.html
```

### 3. Setup Jenkins Server (Ubuntu EC2)

Connect to your **Jenkins Server** Ubuntu instance:

```bash
ssh -i your-key.pem ubuntu@your-jenkins-server-public-ip
```

#### Install Java (Required for Jenkins)

```bash
# Update system packages
sudo apt update -y

# Install OpenJDK 11 (required for Jenkins)
sudo apt install openjdk-11-jdk -y

# Verify Java installation
java -version
```

#### Install Jenkins on Ubuntu

```bash
# Add Jenkins repository key
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

# Add Jenkins repository
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update package list
sudo apt update -y

# Install Jenkins
sudo apt install jenkins -y

# Start Jenkins service
sudo systemctl start jenkins

# Enable Jenkins to start on boot
sudo systemctl enable jenkins

# Check Jenkins status
sudo systemctl status jenkins

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 4. Test Both Servers

#### Test Web Server
Open your browser and navigate to:
```
http://your-web-server-public-ip
```
You should see "Pokemon Store Server Ready!" message.

#### Test Jenkins Server
Open your browser and navigate to:
```
http://your-jenkins-server-public-ip:8080
```
You should see Jenkins setup wizard.

---

## Jenkins Configuration

### 1. Complete Jenkins Initial Setup

#### Access Jenkins Web Interface

Navigate to:
```
http://your-jenkins-server-public-ip:8080
```

1. **Enter the initial admin password** (from previous step)
2. **Install suggested plugins** (recommended)
3. **Create first admin user**
4. **Configure Jenkins URL** (use your Jenkins server public IP)
5. **Start using Jenkins**

### 2. Install Required Jenkins Plugins

Go to **Manage Jenkins** → **Manage Plugins** → **Available**

Install these plugins:
* **Git Plugin** (for repository integration)
* **SSH Agent Plugin** (for SSH key management)
* **Pipeline Plugin** (for Jenkinsfile support)
* **GitHub Integration Plugin** (for webhook support)
* **Generic Webhook Trigger Plugin** (for automatic builds)

### 3. Configure Jenkins to Use Built-in Agent

Since everything runs on Ubuntu EC2, we'll use Jenkins built-in agent:

1. Go to **Manage Jenkins** → **Manage Nodes and Clouds**
2. Click on **Built-In Node**
3. **Configure** → Set **Number of executors** to 2
4. **Labels:** `linux-agent ubuntu-agent`
5. **Save**

### 4. Add EC2 SSH Credentials to Jenkins

1. Go to **Manage Jenkins** → **Manage Credentials**
2. Click **(global)** → **Add Credentials**
3. **Kind:** SSH Username with private key
4. **ID:** `ec2-ssh`
5. **Username:** `ubuntu` (for Ubuntu EC2 instances)
6. **Private Key:** Paste your EC2 private key content (same key used for both servers)
7. **Description:** `Ubuntu EC2 SSH Key for deployment`
8. Click **OK**

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

#### Configure Ubuntu Firewall for Jenkins

On your Jenkins Ubuntu server:
```bash
# Enable UFW firewall
sudo ufw enable

# Allow SSH access
sudo ufw allow ssh

# Allow Jenkins port
sudo ufw allow 8080

# Check firewall status
sudo ufw status
```

#### Verify Jenkins Accessibility

Test that Jenkins is accessible from internet:
```bash
# From your local machine or another server
curl -I http://your-jenkins-server-public-ip:8080

# Should return HTTP 200 or redirect to login page
```

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

Before running, update the `Jenkinsfile` with your Web Server details:

```groovy
environment {
    EC2_USER = "ubuntu"                    # Ubuntu EC2 default user
    EC2_HOST = "your-web-server-public-ip" # Replace with your Web Server public IP
    DEPLOY_PATH = "/var/www/html"          # Nginx default document root
    GIT_REPO = "https://github.com/Ayush-Singh986/pokemon-anime-store.git"
}
```

**Important:** Use your **Web Server** public IP, not the Jenkins server IP!

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

### Issue: Jenkins Cannot Connect to Web Server

**Solution:**
```bash
# Verify SSH key permissions on Jenkins server
chmod 400 your-key.pem

# Test SSH connection manually from Jenkins server
ssh -i your-key.pem ubuntu@your-web-server-ip

# Check Web Server security group allows SSH from Jenkins server IP
# Ensure Jenkins server IP is allowed in Web Server security group on port 22
```

### Issue: Files Not Deploying

**Solution:**
```bash
# Check deployment directory permissions on Web Server
ssh -i your-key.pem ubuntu@your-web-server-ip "ls -la /var/www/html"

# Ensure Nginx is running on Web Server
ssh -i your-key.pem ubuntu@your-web-server-ip "sudo systemctl status nginx"

# Check Jenkins console output for rsync errors
```

### Issue: Website Shows 403 Forbidden

**Solution:**
```bash
# Fix file permissions on Ubuntu Web Server
ssh -i your-key.pem ubuntu@your-web-server-ip "
    sudo chmod -R 755 /var/www/html
    sudo chown -R ubuntu:ubuntu /var/www/html
    sudo systemctl restart nginx
"
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
