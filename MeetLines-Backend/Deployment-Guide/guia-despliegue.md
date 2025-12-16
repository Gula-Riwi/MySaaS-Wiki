# Guía de Despliegue - MeetLines Backend

## Descripción General

Esta guía cubre el despliegue de MeetLines Backend en un servidor VPS usando Docker Compose, Nginx y certificados SSL con Let's Encrypt.

**Entorno de Destino:**
- Servidor VPS Linux (Ubuntu 20.04+)
- Docker & Docker Compose
- Nginx como reverse proxy
- PostgreSQL en contenedor Docker
- Certificados SSL con Certbot

---

## Requisitos Previos

### Servidor VPS

- Ubuntu 20.04 LTS o posterior
- Mínimo 2 GB RAM
- Mínimo 20 GB SSD
- Acceso SSH como root o sudoer
- Dominio configurado (ej: `api.meet-lines.com`)

### Herramientas Locales

```bash
# Verificar acceso SSH
ssh -i /path/to/key.pem user@your-vps-ip

# Verificar SSH está disponible
which ssh
```

---

## Paso 1: Preparar el Servidor VPS

### 1.1 Actualizar el sistema

```bash
# Conectarse al VPS
ssh -i /path/to/key.pem ubuntu@your-vps-ip

# Actualizar paquetes
sudo apt update
sudo apt upgrade -y

# Instalar utilidades básicas
sudo apt install -y \
  curl \
  wget \
  git \
  htop \
  net-tools \
  nano \
  tmux
```

### 1.2 Crear usuario de despliegue

```bash
# Crear usuario
sudo adduser meetlines-app

# Otorgar permisos sudo
sudo usermod -aG sudo meetlines-app

# Cambiar a nuevo usuario
su - meetlines-app

# Crear directorio de trabajo
mkdir -p ~/projects/meetlines-backend
cd ~/projects/meetlines-backend
```

### 1.3 Configurar firewall

```bash
# Permitir SSH
sudo ufw allow 22/tcp

# Permitir HTTP
sudo ufw allow 80/tcp

# Permitir HTTPS
sudo ufw allow 443/tcp

# Habilitar firewall
sudo ufw enable

# Verificar estado
sudo ufw status
```

---

## Paso 2: Instalar Docker y Docker Compose

### 2.1 Instalar Docker

```bash
# Descargar script de instalación
curl -fsSL https://get.docker.com -o get-docker.sh

# Ejecutar instalador
sudo sh get-docker.sh

# Agregar usuario al grupo docker
sudo usermod -aG docker meetlines-app

# Aplicar cambios (logout/login o)
newgrp docker

# Verificar instalación
docker --version
docker compose version
```

### 2.2 Habilitar Docker al iniciar

```bash
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker
```

---

## Paso 3: Instalar Nginx y Certbot

### 3.1 Instalar Nginx

```bash
# Actualizar repositorios
sudo apt update

# Instalar Nginx
sudo apt install -y nginx

# Iniciar Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Verificar
sudo systemctl status nginx
```

### 3.2 Instalar Certbot

```bash
# Instalar Certbot
sudo apt install -y certbot python3-certbot-nginx

# Verificar
certbot --version
```

---

## Paso 4: Clonar y Configurar la Aplicación

### 4.1 Clonar repositorio

```bash
cd ~/projects/meetlines-backend

# Clonar repositorio
git clone https://github.com/Gula-Riwi/MeetLines-Backend.git .

# Verificar rama main
git status
```

### 4.2 Crear archivo `.env` de producción

```bash
# Crear archivo
nano .env.production

# O copiar si existe plantilla
cp .env.example .env.production
```

Contenido del `.env.production`:

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

# Servicios Externos
WHATSAPP_API_URL=https://graph.instagram.com/v18.0
WHATSAPP_API_TOKEN=your_whatsapp_token_prod
WHATSAPP_BUSINESS_ACCOUNT_ID=your_business_account_id_prod

TWILIO_ACCOUNT_SID=your_twilio_sid_prod
TWILIO_AUTH_TOKEN=your_twilio_token_prod
TWILIO_PHONE_NUMBER=your_twilio_phone_prod

SENDGRID_API_KEY=your_sendgrid_key_prod
SENDGRID_FROM_EMAIL=noreply@meet-lines.com

# Configuración de la Aplicación
CORS_ORIGINS=https://app.meet-lines.com
LOG_LEVEL=Warning
DATABASE_MIGRATION_ON_STARTUP=true

# Configuración de Despliegue
DOCKER_REGISTRY=your_docker_registry
DOCKER_IMAGE=meetlines-backend
DOCKER_TAG=latest
```

### 4.3 Crear archivo `docker-compose.prod.yml`

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

## Paso 5: Configurar Nginx

### 5.1 Crear configuración de Nginx

```bash
sudo nano /etc/nginx/sites-available/api.meet-lines.com
```

Contenido:

```nginx
upstream meetlines_api {
    server api:5000;
}

server {
    listen 80;
    server_name api.meet-lines.com;
    
    # Redireccionar a HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.meet-lines.com;
    
    # Certificados SSL
    ssl_certificate /etc/letsencrypt/live/api.meet-lines.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.meet-lines.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    # Headers de seguridad
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Compresión
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
    gzip_min_length 1000;
    
    # Proxy a la API
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

### 5.2 Habilitar sitio en Nginx

```bash
# Crear enlace simbólico
sudo ln -s /etc/nginx/sites-available/api.meet-lines.com \
  /etc/nginx/sites-enabled/api.meet-lines.com

# Eliminar sitio default
sudo rm /etc/nginx/sites-enabled/default

# Verificar configuración
sudo nginx -t

# Recargar Nginx
sudo systemctl reload nginx
```

---

## Paso 6: Obtener Certificados SSL con Let's Encrypt

### 6.1 Obtener certificado

```bash
sudo certbot certonly --standalone \
  -d api.meet-lines.com \
  --email admin@meet-lines.com \
  --agree-tos \
  --non-interactive

# Verificar
sudo ls -la /etc/letsencrypt/live/api.meet-lines.com/
```

### 6.2 Configurar renovación automática

```bash
# Crear script de renovación
sudo nano /etc/letsencrypt/renewal-hook.sh

# Contenido:
#!/bin/bash
docker restart meetlines-nginx-prod

# Hacer ejecutable
sudo chmod +x /etc/letsencrypt/renewal-hook.sh

# Configurar cron para renovar cada mes
sudo crontab -e

# Agregar línea:
0 3 1 * * certbot renew --quiet --renew-hook=/etc/letsencrypt/renewal-hook.sh
```

---

## Paso 7: Desplegar la Aplicación

### 7.1 Construir e iniciar contenedores

```bash
cd ~/projects/meetlines-backend

# Construir imagen de la API
docker compose -f docker-compose.prod.yml build

# Iniciar servicios
docker compose -f docker-compose.prod.yml up -d

# Verificar estado
docker compose -f docker-compose.prod.yml ps

# Ver logs
docker compose -f docker-compose.prod.yml logs -f
```

### 7.2 Aplicar migraciones de BD

```bash
# Ejecutar migraciones dentro del contenedor
docker compose -f docker-compose.prod.yml exec api \
  dotnet ef database update --project MeetLines.Infrastructure

# Verificar
docker compose -f docker-compose.prod.yml exec postgres \
  psql -U postgres -d meetlines_prod -c "\dt"
```

---

## Paso 8: Verificar Despliegue

### 8.1 Health Check

```bash
# Verificar API
curl -k https://api.meet-lines.com/api/health/health

# Respuesta esperada:
# {"status":"healthy","timestamp":"2025-01-20T14:30:00Z"}
```

### 8.2 Acceder a Swagger

```
https://api.meet-lines.com/swagger
```

### 8.3 Verificar logs

```bash
# Logs de la API
docker compose -f docker-compose.prod.yml logs api

# Logs de PostgreSQL
docker compose -f docker-compose.prod.yml logs postgres

# Logs de Nginx
sudo docker logs meetlines-nginx-prod
```

---

## Mantenimiento y Monitoreo

### Monitoreo de contenedores

```bash
# Ver uso de recursos
docker stats

# Ver logs en tiempo real
docker compose -f docker-compose.prod.yml logs -f --tail 100

# Inspeccionar contenedor
docker inspect meetlines-api-prod
```

### Backup de base de datos

```bash
# Crear backup
docker compose -f docker-compose.prod.yml exec postgres \
  pg_dump -U postgres meetlines_prod > backup.sql

# Restaurar desde backup
docker compose -f docker-compose.prod.yml exec -T postgres \
  psql -U postgres meetlines_prod < backup.sql
```

### Actualizaciones

```bash
# Descargar últimos cambios
git pull origin main

# Reconstruir imagen
docker compose -f docker-compose.prod.yml build

# Reiniciar servicios (con zero downtime)
docker compose -f docker-compose.prod.yml up -d

# Aplicar nuevas migraciones
docker compose -f docker-compose.prod.yml exec api \
  dotnet ef database update --project MeetLines.Infrastructure
```

---

## Solución de Problemas

### API no responde

```bash
# Verificar si el contenedor está corriendo
docker ps | grep meetlines-api

# Ver logs de error
docker compose -f docker-compose.prod.yml logs api --tail 50

# Reiniciar contenedor
docker compose -f docker-compose.prod.yml restart api
```

### Base de datos no conecta

```bash
# Verificar conectividad PostgreSQL
docker compose -f docker-compose.prod.yml exec postgres \
  pg_isready -U postgres

# Verificar logs
docker compose -f docker-compose.prod.yml logs postgres
```

### SSL no funciona

```bash
# Verificar certificado
sudo certbot certificates

# Renovar manualmente
sudo certbot renew --dry-run

# Verificar archivo Nginx
sudo nginx -t
```

---

## Documentación de Referencia

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Let's Encrypt / Certbot](https://certbot.eff.org/)
- [ASP.NET Core Docker](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/docker/)

---

## Próximos Pasos

1. ✅ Servidor configurado y asegurado
2. ✅ Certificado SSL instalado
3. ✅ API desplegada en producción
4. 📊 Configurar monitoreo y alertas
5. 📝 Documentar procedimientos operacionales
6. 🔄 Establecer plan de backup y recuperación

¡Despliegue completado con éxito!
