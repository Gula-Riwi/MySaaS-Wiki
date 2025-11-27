---
title: vps_documentation
description: 
published: true
date: 2025-11-27T20:41:19.822Z
tags: 
editor: markdown
dateCreated: 2025-11-26T22:37:15.897Z
---

# VPS Documentation

## 1. Assigning User Permissions

Root permissions were assigned to each team member using the following commands:

```bash
adduser nombre_integrante

sudo usermod -aG sudo nombre_integrante
```

**Permissions verification:**
To verify the permissions of each user, connect via SSH and run:

```bash
ssh nombre_integrante@<IP_del_servidor>
sudo whoami
```

If the result is `root`, the permissions are correctly configured.

---

## 2. Swap Memory and Swappiness Configuration

A swap cushion for the VPS in case the RAM becomes saturated.

```bash
# 1. Create a 2GB swap file
sudo fallocate -l 2G /swapfile

# 2. Set permissions only for root
sudo chmod 600 /swapfile

# 3. Format the file as swap
sudo mkswap /swapfile

# 4. Enable swap temporarily
sudo swapon /swapfile

# 5. Make swap persistent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# 6. Set swappiness to 10 (temporary)
sudo sysctl vm.swappiness=10

# 7. Make swappiness permanent
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf

# 8. Reboot the system to apply changes
sudo reboot
```

**Verification:**
```bash
# Verify that swap is active
sudo swapon --show

# Check the current swappiness value
cat /proc/sys/vm/swappiness
```

---

## 3. Installing Docker and Docker Compose

```bash
# 1. Download Docker install script
curl -fsSL https://get.docker.com -o get-docker.sh

# 2. Run the install script
sudo sh get-docker.sh

# 3. Install Docker Compose Plugin
sudo apt install docker-compose-plugin -y
```

**Grant Docker permissions to users:**
To avoid using `sudo` for every Docker command:

```bash
sudo usermod -aG docker nombre_integrante
```

**Nota:** El usuario debe cerrar sesión y volver a iniciar sesión para que los cambios surtan efecto.

**Verification:**
```bash
docker --version
docker compose version
```

---

## 4. Installing Nginx and Certbot

```bash
# Install Nginx
sudo apt update
sudo apt install nginx -y

# Install Certbot for SSL certificates
sudo apt install certbot python3-certbot-nginx -y
```

**Verification:**
```bash
sudo systemctl status nginx
nginx -v
certbot --version
```

---

## 5. Optimized Deployments

### 5.1. Create a Shared Docker Network

Create a shared Docker network for the entire project:

```bash
docker network create meetlines-agent
```

Esta red permite que los contenedores de diferentes servicios se comuniquen entre sí.

### 5.2. Recommended Project Folder Structure

Recommended organization for different services:

```
/root/projects/
├── wiki/
│   └── docker-compose.yml        # Wiki.js y PostgreSQL
│
├── monitoring/
│   ├── docker-compose.yml        # Grafana, Prometheus, Alertmanager
│   └── prometheus/
│       └── prometheus.yml        # Prometheus configuration
│
├── tools/
│   └── docker-compose.yml        # Portainer (gestión de contenedores)
│
└── app/
    ├── docker-compose.yml        # Servicios de la aplicación principal
    ├── backend/
    │   ├── Dockerfile
    │   └── ...                   # Código fuente del backend
    └── frontend/
        ├── Dockerfile
        └── ...                   # Código fuente del frontend
```

**Advantages of this structure:**
- Separation of services by responsibility
- Easier maintenance and scalability
- Each service can restart independently

---

## 6. Nginx Configuration for Wiki.js

### 6.1. Create the configuration file

```bash
sudo nano /etc/nginx/sites-available/docs.gula.crudzaso.com
```

**Configuration file content:**

```nginx
server {
    listen 80;
    server_name docs.gula.crudzaso.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 6.2. Activate the configuration

```bash
# Create symbolic link in sites-enabled
sudo ln -s /etc/nginx/sites-available/docs.gula.crudzaso.com /etc/nginx/sites-enabled/

# Verify configuration syntax
sudo nginx -t

# Reload Nginx
sudo systemctl restart nginx
```

### 6.3. Configure SSL with Certbot

```bash
# Obtain SSL certificate automatically
sudo certbot --nginx -d docs.gula.crudzaso.com

# Certbot will automatically modify the Nginx configuration to use HTTPS
```

### 6.4. Verification

```bash
# Verify that the domain resolves correctly
ping docs.gula.crudzaso.com

# Verify the SSL certificate
curl -I https://docs.gula.crudzaso.com
```

---

### Nginx

```bash
# Verify Nginx configuration syntax
sudo nginx -t

# Reload configuration without stopping the service
sudo systemctl reload nginx

# Restart Nginx
sudo systemctl restart nginx

# Tail Nginx logs
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log
```

### System

```bash
# Show memory and swap usage
free -h

# Show disk usage
df -h

# Show top resource consuming processes
htop

# Show open ports
sudo netstat -tulpn
```

