# Documentación VPS

## 1. Asignación de Permisos de Usuario

Se asignaron permisos root a cada integrante del equipo con los siguientes comandos:

```bash
adduser nombre_integrante

sudo usermod -aG sudo nombre_integrante
```

**Verificación de permisos:**
Para verificar los permisos de cada usuario, conectarse via SSH y ejecutar:

```bash
ssh nombre_integrante@<IP_del_servidor>
sudo whoami
```

Si el resultado es `root`, los permisos están correctamente configurados.

---

## 2. Configuración de Memoria Swap y Swappiness

Colchón de memoria para el VPS en caso de estar saturado por la memoria RAM.

```bash
# 1. Crear el archivo swap de 2GB
sudo fallocate -l 2G /swapfile

# 2. Ajustar permisos solo para root
sudo chmod 600 /swapfile

# 3. Formatear el archivo como swap
sudo mkswap /swapfile

# 4. Activar el swap temporalmente
sudo swapon /swapfile

# 5. Hacer el swap persistente
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# 6. Ajustar swappiness a 10 (temporalmente)
sudo sysctl vm.swappiness=10

# 7. Hacer el swappiness permanente
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf

# 8. Reiniciar el sistema para aplicar todos los cambios
sudo reboot
```

**Verificación:**
```bash
# Verificar que el swap esté activo
sudo swapon --show

# Verificar el valor de swappiness
cat /proc/sys/vm/swappiness
```

---

## 3. Instalación de Docker y Docker Compose

```bash
# 1. Descargar el script de instalación de Docker
curl -fsSL https://get.docker.com -o get-docker.sh

# 2. Ejecutar el script de instalación
sudo sh get-docker.sh

# 3. Instalar Docker Compose Plugin
sudo apt install docker-compose-plugin -y
```

**Dar permisos Docker a usuarios:**
Para evitar tener que usar `sudo` con cada comando de Docker:

```bash
sudo usermod -aG docker nombre_integrante
```

**Nota:** El usuario debe cerrar sesión y volver a iniciar sesión para que los cambios surtan efecto.

**Verificación:**
```bash
docker --version
docker compose version
```

---

## 4. Instalación de Nginx y Certbot

```bash
# Instalar Nginx
sudo apt update
sudo apt install nginx -y

# Instalar Certbot para certificados SSL
sudo apt install certbot python3-certbot-nginx -y
```

**Verificación:**
```bash
sudo systemctl status nginx
nginx -v
certbot --version
```

---

## 5. Despliegues Optimizados

### 5.1. Creación de Red Compartida

Crear una red Docker compartida para todo el proyecto:

```bash
docker network create mysaas-agent
```

Esta red permite que los contenedores de diferentes servicios se comuniquen entre sí.

### 5.2. Estructura de Carpetas del Proyecto

Organización recomendada para los diferentes servicios:

```
/root/projects/
├── wiki/
│   └── docker-compose.yml        # Wiki.js y PostgreSQL
│
├── monitoring/
│   ├── docker-compose.yml        # Grafana, Prometheus, Alertmanager
│   └── prometheus/
│       └── prometheus.yml        # Configuración de Prometheus
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

**Ventajas de esta estructura:**
- Separación de servicios por responsabilidad
- Fácil mantenimiento y escalabilidad
- Cada servicio puede reiniciarse independientemente

---

## 6. Configuración de Nginx para Wiki.js

### 6.1. Crear el archivo de configuración

```bash
sudo nano /etc/nginx/sites-available/docs.gula.crudzaso.com
```

**Contenido del archivo:**

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

### 6.2. Activar la configuración

```bash
# Crear enlace simbólico en sites-enabled
sudo ln -s /etc/nginx/sites-available/docs.gula.crudzaso.com /etc/nginx/sites-enabled/

# Verificar sintaxis de configuración
sudo nginx -t

# Recargar Nginx
sudo systemctl restart nginx
```

### 6.3. Configurar SSL con Certbot

```bash
# Obtener certificado SSL automáticamente
sudo certbot --nginx -d docs.gula.crudzaso.com

# Certbot modificará automáticamente la configuración de Nginx para usar HTTPS
```

### 6.4. Verificación

```bash
# Verificar que el dominio resuelve correctamente
ping docs.gula.crudzaso.com

# Verificar el certificado SSL
curl -I https://docs.gula.crudzaso.com
```

---

## 7. Configuración de Bases de Datos Centralizadas

### 7.1. Estructura del servicio de bases de datos

Crear directorio para las bases de datos centralizadas:

```bash
mkdir -p /projects/databases
cd /projects/databases
```

### 7.2. Archivo docker-compose.yml para PostgreSQL y Redis

```yaml
version: "3.5"

services:
  central-postgres:
    image: postgres:15-alpine
    container_name: central-postgres
    hostname: central-postgres
    environment:
      - POSTGRES_PASSWORD=${GLOBAL_POSTGRES_PASS}
    ports:
      - "5432:5432"
    volumes:
      - ./postgres_data:/var/lib/postgresql/data
    networks:
      - mysaas-agent
    restart: unless-stopped

    deploy:
      resources:
        limits:
          memory: 1280m
    logging:
      options:
        max-size: "20m"
        max-file: "5"

  central-redis:
    image: redis:alpine
    container_name: central-redis
    hostname: central-redis
    command: redis-server --requirepass ${GLOBAL_REDIS_PASS}
    volumes:
      - ./redis_data:/data
    networks:
      - mysaas-agent
    restart: unless-stopped

    deploy:
      resources:
        limits:
          memory: 256m
    logging:
      options:
        max-size: "20m"
        max-file: "5"

networks:
  mysaas-agent:
    external: true
```

### 7.3. Configuración de variables de entorno

Crear archivo `.env` en el directorio `/projects/databases/`:

```bash
# Variables de base de datos
GLOBAL_POSTGRES_PASS=tu_password_seguro_postgres
GLOBAL_REDIS_PASS=tu_password_seguro_redis
```

**Asegurar el archivo .env:**
```bash
chmod 600 .env
```

### 7.4. Iniciar los servicios de base de datos

```bash
# Desde el directorio /root/projects/databases/
docker compose up -d

# Verificar que los contenedores estén ejecutándose
docker ps

# Ver logs de PostgreSQL
docker logs central-postgres

# Ver logs de Redis
docker logs central-redis
```

### 7.5. Conexión a las bases de datos

**PostgreSQL:**
- **Host:** `central-postgres` (desde otros contenedores) o `localhost` (desde el host)
- **Puerto:** `5432`
- **Usuario:** `postgres`
- **Contraseña:** Valor de `GLOBAL_POSTGRES_PASS`

**Redis:**
- **Host:** `central-redis` (desde otros contenedores) o `localhost` (desde el host)
- **Puerto:** `6379` (puerto por defecto)
- **Contraseña:** Valor de `GLOBAL_REDIS_PASS`

## Comandos Útiles de Administración

### Nginx

```bash
# Verificar que las sintaxys de las configuraciones esten correctamente
sudo nginx -t

# Recargar configuración sin detener el servicio
sudo systemctl reload nginx

# Reiniciar Nginx
sudo systemctl restart nginx

# Ver logs de Nginx
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log
```

### Sistema

```bash
# Ver uso de memoria y swap
free -h

# Ver uso de disco
df -h

# Ver procesos que más consumen recursos
htop

# Ver puertos en uso
sudo netstat -tulpn
```

