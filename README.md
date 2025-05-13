# Deploying a Next.js Application on Ubuntu Server

This guide provides step-by-step instructions for setting up an Ubuntu server to deploy a Next.js application. It covers installing and configuring Nginx as a reverse proxy, setting up Node.js and PM2, and deploying your Next.js application.

## Prerequisites

* An Ubuntu server.
* Basic knowledge of Linux commands.
* (Optional) A domain name.

## Step 1: Prepare the Ubuntu Server

These commands update the system, install necessary build tools, and set up Node.js and PM2.

```bash
# 1. Update your package list
sudo apt update

# 2. Install build essentials and curl (required for building Node.js and using NVM)
sudo apt install -y build-essential curl

# 3. Install NVM (Node Version Manager)
curl -o- [https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh](https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh) | bash

# 4. Load NVM into your shell session
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# 5. Install the latest LTS version of Node.js
nvm install --lts

# 6. Use the latest LTS version by default
nvm alias default 'lts/*'

# 7. Install PM2 globally (for process management)
npm install -g pm2

# 8. Verify installations
node -v  # Check Node.js version
npm -v   # Check npm version
pm2 -v    # Check PM2 version
Step 2: Install and Configure NginxNginx will act as a reverse proxy, forwarding requests to your Next.js application.# Install Nginx
sudo apt install -y nginx

# Configure Nginx as a reverse proxy. Create a new config file:
sudo nano /etc/nginx/sites-available/nextjs
Paste the following configuration into the nextjs file.  Important: Replace your_domain.com with your actual domain name if you have one, otherwise, you can use the server's IP address.server {
    listen 80;
    server_name your_domain.com; # Replace with your domain or server IP

    location / {
        proxy_pass http://localhost:3000;  # Next.js default port
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
Save the file and create a symbolic link to enable the configuration:sudo ln -s /etc/nginx/sites-available/nextjs /etc/nginx/sites-enabled/

# Test the Nginx configuration
sudo nginx -t

# Restart Nginx to apply the changes
sudo systemctl restart nginx
Step 3: (Optional) Set Up a Domain and SSLIf you have a domain name, point it to your server's IP address using your DNS provider's control panel.  This step is not required if you are using the server's IP directly.For SSL (HTTPS) encryption, use Certbot:sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your_domain.com # Replace with your domain name
Certbot will automatically configure SSL for your domain, obtaining a certificate from Let's Encrypt.Step 4: Deploy Your Next.js ApplicationThese steps assume your Next.js project is located at /home/ubuntu/internal-app.  Adjust the path if your project is in a different location.# 1. Go to your project directory
cd /home/ubuntu/internal-app

# 2. Install project dependencies
npm install

# 3. Build the Next.js application for production
npm run build

# 4. Start the Next.js application using PM2
pm2 start npm --name "nextjs-app" -- start

# 5.  Make PM2 start on boot
pm2 save
pm2 startup
