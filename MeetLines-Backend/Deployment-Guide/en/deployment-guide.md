# Deployment Guide - MeetLines Backend

## Overview

This guide covers deploying MeetLines Backend to a VPS using Docker Compose, Nginx, and SSL certificates with Let's Encrypt.

**Target Environment:**
- Linux VPS (Ubuntu 20.04+)
- Docker & Docker Compose
- Nginx as reverse proxy
- PostgreSQL in Docker container
- SSL certificates with Certbot

---

## Prerequisites

### VPS Server

- Ubuntu 20.04 LTS or later
- Minimum 2 GB RAM
- Minimum 20 GB SSD
- SSH access as root or sudoer
- Domain configured (e.g., `api.meet-lines.com`)

### Local Tools

```bash
# Verify SSH access
ssh -i /path/to/key.pem user@your-vps-ip

# Verify SSH is available
which ssh
```

---

## Step 1: Prepare VPS Server

### 1.1 Update system

```bash
# Connect to VPS
ssh -i /path/to/key.pem ubuntu@your-vps-ip

# Update packages
sudo apt update
sudo apt upgrade -y

# Install basic utilities
sudo apt install -y \
  curl \
  wget \
  git \
  htop \
  net-tools \
  nano \
  tmux
```

### 1.2 Create deployment user

```bash
# Create user
sudo adduser meetlines-app

# Grant sudo permissions
sudo usermod -aG sudo meetlines-app

# Switch to new user
su - meetlines-app

# Create work directory
mkdir -p ~/projects/meetlines-backend
cd ~/projects/meetlines-backend
```

### 1.3 Configure firewall

```bash
# Allow SSH
sudo ufw allow 22/tcp

# Allow HTTP
sudo ufw allow 80/tcp

# Allow HTTPS
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw enable

# Verify status
sudo ufw status
```

---

## Step 2: Install Docker and Docker Compose

### 2.1 Install Docker

```bash
# Download installation script
curl -fsSL https://get.docker.com -o get-docker.sh

# Run installer
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker meetlines-app

# Apply changes (logout/login or)
newgrp docker

# Verify installation
docker --version
docker compose version
```

### 2.2 Enable Docker on startup

```bash
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker
```

---

## Step 3: Install Nginx and Certbot

### 3.1 Install Nginx

```bash
# Update repositories
sudo apt update

# Install Nginx
sudo apt install -y nginx

# Start Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify
sudo systemctl status nginx
```

### 3.2 Install Certbot

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Verify
certbot --version
```

---

## Step 4: Clone and Configure Application

### 4.1 Clone repository

```bash
cd ~/projects/meetlines-backend

# Clone repository
git clone https://github.com/Gula-Riwi/MeetLines-Backend.git .

# Verify main branch
git status
```

### 4.2 Create production `.env` file

```bash
# Create file
nano .env.production

# Or copy if template exists
cp .env.example .env.production
```

Content of `.env.production`:

```env
# PostgreSQL
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_DB=meetlines_prod
POSTGRES_USER=postgres
POSTGRES_PASSWORD=CHANGE_THIS_TO_SECURE_PASSWORD
POSTGRES_CONNECTION_STRING=Server=postgres;Port=5432;Database=meetlines_prod;User Id=postgres;Password=CHANGE_THIS_TO_SECURE_PASSWORD;

# ASP.NET Core
ASPNETCORE_ENVIRONMENT=Production
ASPNETCORE_URLS=http://0.0.0.0:5000
ASPNETCORE_HTTPS_PORT=5001
JWT_KEY=GENERATE_A_LONG_RANDOM_STRING_MIN_256_BITS
JWT_ISSUER=meetlines
JWT_AUDIENCE=meetlines-api

# OAuth Providers (Production credentials)
DISCORD_CLIENT_ID=your_production_discord_id
DISCORD_CLIENT_SECRET=your_production_discord_secret
DISCORD_REDIRECT_URI=https://api.meet-lines.com/api/auth/oauth/discord/callback

FACEBOOK_APP_ID=your_production_facebook_id
FACEBOOK_APP_SECRET=your_production_facebook_secret
FACEBOOK_REDIRECT_URI=https://api.meet-lines.com/api/auth/oauth/facebook/callback

GOOGLE_CLIENT_ID=your_production_google_id
GOOGLE_CLIENT_SECRET=your_production_google_secret
GOOGLE_REDIRECT_URI=https://api.meet-lines.com/api/auth/oauth/google/callback

# External Services
WHATSAPP_API_URL=https://graph.instagram.com/v18.0
WHATSAPP_API_TOKEN=your_whatsapp_token_prod
WHATSAPP_BUSINESS_ACCOUNT_ID=your_business_account_id_prod

TWILIO_ACCOUNT_SID=your_twilio_sid_prod
TWILIO_AUTH_TOKEN=your_twilio_token_prod
TWILIO_PHONE_NUMBER=your_twilio_phone_prod

SENDGRID_API_KEY=your_sendgrid_key_prod
SENDGRID_FROM_EMAIL=noreply@meet-lines.com

# Application Configuration
CORS_ORIGINS=https://app.meet-lines.com
LOG_LEVEL=Warning
DATABASE_MIGRATION_ON_STARTUP=true

# Deployment Configuration
DOCKER_REGISTRY=your_docker_registry
DOCKER_IMAGE=meetlines-backend
DOCKER_TAG=latest
```

### 4.3 Create `docker-compose.prod.yml` file

```bash
cat > docker-compose.prod.yml << 'EOF'
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: meetlines-db-prod
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - meetlines-network
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  api:
    image: mcr.microsoft.com/dotnet/aspnet:8.0
    container_name: meetlines-api-prod
    build:
      context: .
      dockerfile: MeetLines.API/Dockerfile
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_URLS=http://0.0.0.0:5000
      - POSTGRES_CONNECTION_STRING=${POSTGRES_CONNECTION_STRING}
      - JWT_KEY=${JWT_KEY}
      - JWT_ISSUER=${JWT_ISSUER}
      - JWT_AUDIENCE=${JWT_AUDIENCE}
      - WHATSAPP_API_TOKEN=${WHATSAPP_API_TOKEN}
      - SENDGRID_API_KEY=${SENDGRID_API_KEY}
    ports:
      - "5000:5000"
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - meetlines-network
    restart: always
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/api/health/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  nginx:
    image: nginx:alpine
    container_name: meetlines-nginx-prod
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.prod.conf:/etc/nginx/nginx.conf:ro
      - /etc/letsencrypt:/etc/letsencrypt:ro
      - nginx-cache:/var/cache/nginx
    depends_on:
      - api
    networks:
      - meetlines-network
    restart: always

networks:
  meetlines-network:
    driver: bridge

volumes:
  postgres-data:
  nginx-cache:
EOF
```

---

## Step 5: Configure Nginx

### 5.1 Create Nginx configuration

```bash
sudo nano /etc/nginx/sites-available/api.meet-lines.com
```

Content:

```nginx
upstream meetlines_api {
    server api:5000;
}

server {
    listen 80;
    server_name api.meet-lines.com;
    
    # Redirect to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.meet-lines.com;
    
    # SSL Certificates
    ssl_certificate /etc/letsencrypt/live/api.meet-lines.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.meet-lines.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
    gzip_min_length 1000;
    
    # Proxy to API
    location / {
        proxy_pass http://meetlines_api;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $server_name;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

### 5.2 Enable site in Nginx

```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/api.meet-lines.com \
  /etc/nginx/sites-enabled/api.meet-lines.com

# Remove default site
sudo rm /etc/nginx/sites-enabled/default

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

---

## Step 6: Get SSL Certificates with Let's Encrypt

### 6.1 Obtain certificate

```bash
sudo certbot certonly --standalone \
  -d api.meet-lines.com \
  --email admin@meet-lines.com \
  --agree-tos \
  --non-interactive

# Verify
sudo ls -la /etc/letsencrypt/live/api.meet-lines.com/
```

### 6.2 Configure automatic renewal

```bash
# Create renewal script
sudo nano /etc/letsencrypt/renewal-hook.sh

# Content:
#!/bin/bash
docker restart meetlines-nginx-prod

# Make executable
sudo chmod +x /etc/letsencrypt/renewal-hook.sh

# Configure cron to renew monthly
sudo crontab -e

# Add line:
0 3 1 * * certbot renew --quiet --renew-hook=/etc/letsencrypt/renewal-hook.sh
```

---

## Step 7: Deploy Application

### 7.1 Build and start containers

```bash
cd ~/projects/meetlines-backend

# Build API image
docker compose -f docker-compose.prod.yml build

# Start services
docker compose -f docker-compose.prod.yml up -d

# Verify status
docker compose -f docker-compose.prod.yml ps

# View logs
docker compose -f docker-compose.prod.yml logs -f
```

### 7.2 Apply database migrations

```bash
# Run migrations inside container
docker compose -f docker-compose.prod.yml exec api \
  dotnet ef database update --project MeetLines.Infrastructure

# Verify
docker compose -f docker-compose.prod.yml exec postgres \
  psql -U postgres -d meetlines_prod -c "\dt"
```

---

## Step 8: Verify Deployment

### 8.1 Health Check

```bash
# Verify API
curl -k https://api.meet-lines.com/api/health/health

# Expected response:
# {"status":"healthy","timestamp":"2025-01-20T14:30:00Z"}
```

### 8.2 Access Swagger

```
https://api.meet-lines.com/swagger
```

### 8.3 View logs

```bash
# API logs
docker compose -f docker-compose.prod.yml logs api

# PostgreSQL logs
docker compose -f docker-compose.prod.yml logs postgres

# Nginx logs
sudo docker logs meetlines-nginx-prod
```

---

## Maintenance and Monitoring

### Monitor containers

```bash
# View resource usage
docker stats

# View logs in real-time
docker compose -f docker-compose.prod.yml logs -f --tail 100

# Inspect container
docker inspect meetlines-api-prod
```

### Database backup

```bash
# Create backup
docker compose -f docker-compose.prod.yml exec postgres \
  pg_dump -U postgres meetlines_prod > backup.sql

# Restore from backup
docker compose -f docker-compose.prod.yml exec -T postgres \
  psql -U postgres meetlines_prod < backup.sql
```

### Updates

```bash
# Download latest changes
git pull origin main

# Rebuild image
docker compose -f docker-compose.prod.yml build

# Restart services (with zero downtime)
docker compose -f docker-compose.prod.yml up -d

# Apply new migrations
docker compose -f docker-compose.prod.yml exec api \
  dotnet ef database update --project MeetLines.Infrastructure
```

---

## Troubleshooting

### API not responding

```bash
# Check if container is running
docker ps | grep meetlines-api

# View error logs
docker compose -f docker-compose.prod.yml logs api --tail 50

# Restart container
docker compose -f docker-compose.prod.yml restart api
```

### Database won't connect

```bash
# Verify PostgreSQL connectivity
docker compose -f docker-compose.prod.yml exec postgres \
  pg_isready -U postgres

# View logs
docker compose -f docker-compose.prod.yml logs postgres
```

### SSL not working

```bash
# Check certificate
sudo certbot certificates

# Renew manually
sudo certbot renew --dry-run

# Verify Nginx config
sudo nginx -t
```

---

## Reference Documentation

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Let's Encrypt / Certbot](https://certbot.eff.org/)
- [ASP.NET Core Docker](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/docker/)

---

## Next Steps

1. ✅ Server configured and secured
2. ✅ SSL certificate installed
3. ✅ API deployed to production
4. 📊 Set up monitoring and alerts
5. 📝 Document operational procedures
6. 🔄 Establish backup and recovery plan

Deployment completed successfully!
