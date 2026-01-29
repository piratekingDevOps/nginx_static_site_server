# Static Site Deployment on Remote Linux Server 🚀

## Project Link

https://roadmap.sh/projects/static-site-server

---

This project demonstrates how to deploy a static HTML/CSS site to a remote Linux server (AWS EC2) using **rsync** and **GitHub Actions** for automation.

---

## Table of Contents

* [Project Overview](#project-overview)
* [Prerequisites](#prerequisites)
* [Server Setup](#server-setup)
* [Deployment Automation](#deployment-automation)
* [Testing](#testing)
* [Security Best Practices](#security-best-practices)
* [Resources](#resources)

---

## Project Overview

The goal of this project is to:

1. Set up a remote Linux server (EC2 instance).
2. Configure Nginx to serve a static site.
3. Deploy site updates automatically using GitHub Actions + rsync.
4. Ensure secure access with SSH keys.

By completing this project, you will have a **basic CI/CD pipeline** for static sites.

---

## Prerequisites

* AWS EC2 instance (Amazon Linux 2 recommended)
* SSH access to the server
* Nginx installed
* GitHub repository with your static site
* GitHub Actions enabled

---

## Server Setup

1. **Connect to the server** via SSH:

```bash
ssh -i ~/.ssh/key_name ec2-user@<SERVER_IP>
```

2. **Install Nginx**:

```bash
sudo yum update -y
sudo amazon-linux-extras install nginx1 -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

3. **Create web root**:

```bash
sudo mkdir -p /var/www/html
sudo chown -R ec2-user:ec2-user /var/www/html
sudo chmod -R 755 /var/www/html
```

4. **Create a test index page**:

```bash
echo "<h1>NGINX WORKS 🚀</h1>" > /var/www/html/index.html
sudo systemctl reload nginx
```

5. **Open port 80** in AWS Security Group.

---

## Deployment Automation with GitHub Actions

This project uses **GitHub Actions** to deploy your site automatically whenever you push to `main`.

### 1. Generate SSH key for GitHub Actions

```bash
ssh-keygen -t ed25519 -f github_deploy_key
```

Add the **public key** to:

```bash
~/.ssh/authorized_keys
```

Ensure correct permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

### 2. Add Secrets to GitHub

| Secret Name     | Value                         |
| --------------- | ----------------------------- |
| SSH_PRIVATE_KEY | Contents of github_deploy_key |
| SERVER_IP       | EC2 instance public IP        |
| SSH_USER        | ec2-user                      |
| DEPLOY_PATH     | /var/www/html                 |

---

### 3. GitHub Actions Workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy Static Site

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          ssh-keyscan -H ${{ secrets.SERVER_IP }} >> ~/.ssh/known_hosts

      - name: Deploy with rsync
        run: |
          rsync -avz --delete \
          -e "ssh -i ~/.ssh/id_ed25519 -o StrictHostKeyChecking=no" \
          ./site/ \
          ${{ secrets.SSH_USER }}@${{ secrets.SERVER_IP }}:${{ secrets.DEPLOY_PATH }}
```

---

## Testing

1. **Local server test**:

```bash
curl http://localhost
```

Expected output:

```html
<h1>NGINX WORKS 🚀</h1>
```

2. **Remote test**:

Open browser:

```
http://<SERVER_PUBLIC_IP>
```

---

## Security Best Practices

* Never commit private keys.
* Add `.gitignore` to block keys and sensitive files:

```gitignore
*.pem
*.key
*.pub
id_rsa*
id_ed25519*
.env
```

* Rotate keys if exposed.
* Use SSH over HTTPS or tokens when possible.

---

## Resources

* [Project Link](https://roadmap.sh/projects/static-site-server)
* [HTML5 UP Templates](https://html5up.net/)
* [Start Bootstrap Templates](https://startbootstrap.com/)
* [GitHub Actions Docs](https://docs.github.com/en/actions)
* [AWS EC2 Docs](https://docs.aws.amazon.com/ec2/)

---

**Author:** Your Name
**Date:** January 2026
