
# Pokemon & Anime Store – Jenkins CI/CD Deployment

## Project Overview

This project demonstrates a Jenkins-based CI/CD pipeline for automatically deploying a static web application (Pokemon & Anime Store) to an AWS EC2 Ubuntu server running Nginx.

The deployment process is fully automated using a Jenkinsfile. Any push to the GitHub repository triggers Jenkins to pull the latest code and deploy it to the EC2 web server.

---

## Technology Stack

- Jenkins (Pipeline as Code)
- GitHub (Source Control & Webhooks)
- AWS EC2 (Ubuntu 22.04)
- Nginx Web Server
- SSH and rsync
- HTML / CSS

---

## Repository Structure

```
pokemon-anime-store/
├── index.html        # Pokemon Store homepage
├── anime.html        # Anime characters page
└── Jenkinsfile       # Jenkins CI/CD pipeline
```

---

## Application Description

- **index.html**
  Main landing page for the Pokemon Store.

- **anime.html**
  Anime characters and collectibles page.

Both files are deployed directly to the Nginx document root.

---

## Jenkins Pipeline Overview

The CI/CD pipeline is defined inside the `Jenkinsfile` and performs the following steps:

1. Checkout source code from GitHub
2. Validate required application files
3. Deploy files to EC2 web server using SSH and rsync

No build tools are required because this is a static web application.

---

## Jenkins Job Configuration

- **Job Type:** Pipeline  
- **Pipeline Definition:** Pipeline script from SCM  
- **SCM:** Git  
- **Branch:** main  
- **Script Path:** Jenkinsfile  

### Build Trigger

- GitHub hook trigger for GITScm polling

This allows Jenkins to automatically run the pipeline on every push to the repository.

---

## Jenkins Credentials

Jenkins must be configured with the following credential:

- **Type:** SSH Username with Private Key
- **Username:** ubuntu
- **Purpose:** SSH access to EC2 web server
- **Scope:** Global
- **Credential ID:** Used in Jenkinsfile

---

## Web Server Requirements

The EC2 web server must have:

- Ubuntu 22.04
- Nginx installed and running
- SSH access enabled
- `/var/www/html` directory writable by deployment user

---

## Deployment Flow

```
Developer Push
      ↓
GitHub Repository
      ↓
GitHub Webhook
      ↓
Jenkins Pipeline
      ↓
AWS EC2 (Nginx)
```

---

## Verification

After a successful Jenkins build, open the web server public IP in a browser:

```
http://<EC2-PUBLIC-IP>
```

The Pokemon & Anime Store website should load with the latest changes.

---

## Conclusion

This project showcases a simple and effective Jenkins CI/CD pipeline for deploying a static website on AWS EC2 using industry-standard DevOps practices.
