# CI/CD Pipeline with GitHub Actions, EC2, and Docker

This project demonstrates a complete CI/CD pipeline using GitHub Actions, deploying a Node.js application to Amazon EC2 using Docker.

## Table of Contents

1. [Initial Server Setup](#1-initial-server-setup)
2. [Linting Workflow](#2-linting-workflow)
3. [Deployment Workflow](#3-deployment-workflow)
4. [Nginx and Domain Configuration](#4-nginx-and-domain-configuration)
5. [SSL Certificate Setup](#5-ssl-certificate-setup)
6. [Conclusion](#conclusion)

## 1. Initial Server Setup

### EC2 Instance Creation
1. Launch an EC2 instance on AWS
   - Choose Ubuntu Server 24.04 LTS
   - Select an appropriate instance type (e.g., t2.micro for testing)
   - Configure security group to allow inbound traffic on ports 22 (SSH), 80 (HTTP), and 443 (HTTPS)
   - Create and download a new key pair for SSH access

### Server Configuration
1. Connect to your EC2 instance:
    ```bash
    ssh -i /path/to/your-key.pem ubuntu@your-ec2-public-dns
    ```
2. Update the system and install Docker:
    ```bash
    sudo apt update
    sudo apt upgrade -y
    sudo apt install docker.io -y
    sudo systemctl start docker
    sudo systemctl enable docker
    ```
## 2. Linting Workflow
Create [`.github/workflows/lint.yml`](.github/workflows/lint.yml) to run Biome on every push to the `main` branch.
- This workflow:
   - Triggers on pushes to main and pull requests
   - Sets up Node.js environment
   - Installs dependencies
   - Runs the linting script defined in package.json

## 3. Deployment Workflow
Create [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml) to deploy the Node.js application to the EC2 instance on every push to the `main` branch.
- This workflow:
   - Triggers on pushes to main
   - Sets up Node.js environment
   - Installs dependencies
   - Builds the Docker image
   - Pushes the Docker image to Docker Hub
   - Connects to the EC2 instance
   - Pulls the Docker image from Docker Hub
   - Runs the Docker container
- Add the following secrets to your GitHub repository:
   - `DOCKER_USERNAME`: Your Docker Hub username
   - `DOCKER_PASSWORD`: Your Docker Hub password
   - `EC2_HOST`: Your EC2 public DNS
   - `EC2_USERNAME`: Your EC2 username
   - `EC2_SSH_KEY`: Your EC2 SSH private key

## 4. Nginx and Domain Configuration
1. Install Nginx on the EC2 instance:
    ```bash
    sudo apt install nginx -y
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```
2. Configure Nginx to forward requests to the Node.js application:
    - Create a new Nginx configuration file:
        ```bash
        sudo nano /etc/nginx/nginx.conf
        ```
    - Add the following configuration under the `http` block:
        ```
        server {
            listen 80;
            server_name test.somyabhatt.com(<your-domain>);

            location / {
                proxy_pass http://localhost:3000;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
            }
        }
        ```
    - Save and exit the file
    - Test the Nginx configuration:
        ```bash
        sudo nginx -t
        ```
    - Reload Nginx to apply the changes:
        ```bash
        sudo systemctl reload nginx
        ```
3. Update the domain's DNS settings to point to the EC2 instance's public IP address. In this project, Cloudflare was used to manage the domain's DNS settings. To configure this:
    - Log in to your Cloudflare account and select your domain. 
    - Go to the DNS management section.
    - Create an A record with the following details:
        ```
        Type: A
        Name: test.somyabhatt.com<your-domain>
        Value: <EC2-Public-IP>
        TTL: Auto
        Proxy Status: Proxied
        ```


## 5. SSL Certificate Setup
1. Install Certbot on the EC2 instance:
    ```bash
    sudo apt install certbot -y
    ```
2. Obtain an SSL certificate for the domain:
    ```bash
    sudo certbot --nginx
    ```
3. Follow the interactive prompts to configure the SSL certificate
4. Test the SSL configuration:
    ```bash
    sudo nginx -t
    ```
5. Reload Nginx to apply the changes:
    ```bash
    sudo systemctl reload nginx
    ```

## Conclusion
This project demonstrates a complete CI/CD pipeline using GitHub Actions, deploying a Node.js application to Amazon EC2 using Docker. The application is served over HTTPS using Nginx and an SSL certificate.