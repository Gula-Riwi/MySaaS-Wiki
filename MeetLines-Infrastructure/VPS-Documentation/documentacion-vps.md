---
title: Infraestructura y VPS
description: Documentación técnica completa del entorno de producción.
published: true
tags: infraestructura, vps, docker, seguridad
editor: markdown
---

# Infraestructura y VPS

Documentación técnica del servidor de producción ("Gula-VPS"). Este documento sirve como única fuente de verdad para la administración, despliegue y mantenimiento del sistema.

---

## 1. Especificaciones del Servidor

| Recurso | Detalle |
| :--- | :--- |
| **Sistema Operativo** | Ubuntu 22.04 LTS (Jammy Jellyfish) |
| **CPU** | 2 vCPU |
| **RAM** | 4 GB |
| **Almacenamiento** | 40 GB SSD |
| **Swap** | 2 GB (Configurado en `/swapfile`) |
| **IP Pública** | 46.224..13.157 |

### 1.1. Usuarios y Accesos
*   **Acceso Root**: Deshabilitado por SSH.
*   **Autenticación**: Exclusivamente mediante **SSH Keys** (contraseñas deshabilitadas).
*   **Usuarios Administradores**:
    *   `miguel_arias` 
    *   `omar_andres` 
    *   `jhon_rojas` 
    *   `emmanuel_rendon` 
    *   `jhonatan_morales` 
    *   `juan_sepulveda`
      

---

## 2. Accesos y Seguridad (Hardening)

### 2.1. Protección contra Intrusos (Fail2Ban)
Se utiliza **Fail2Ban** para proteger el puerto SSH y Nginx contra ataques de fuerza bruta.
*   **Jail SSH**: Banea la IP tras 5 intentos fallidos. Tiempo de baneo: 1 hora.

### 2.2. Gestión de Claves SSH
Para dar acceso a un nuevo desarrollador:
1.  Solicitar su clave pública (`id_rsa.pub`).
2.  Loguearse como administrador.
3.  Agregar la clave:
    ```bash
    # En el usuario correspondiente
    echo "ssh-rsa AAAA..." >> ~/.ssh/authorized_keys
    ```

---

## 3. Estructura de Carpetas y Red

La infraestructura se divide en dos ubicaciones principales según su propósito:

### 3.1. Servicios de Infraestructura (`~/projects`)
Ubicados en `/home/miguel_arias/projects` (o `/root/projects` históricamente). Contiene servicios de soporte y automatización.

```
~/projects/
├── automation/       # N8N (Automatización de flujos)
├── databases/        # PostgreSQL Central y Redis
├── monitoring/       # Grafana, Prometheus, Portainer
└── wiki/             # Wiki.js (Documentación)
```

### 3.2. Aplicaciones de Negocio (`/var/www`)
Ubicación estándar para despliegues web automatizados.

```
/var/www/
├── MeetLines-Backend/   # API .NET Core (Desplegado vía Git Pull)
└── MeetLines-Frontend/  # Frontend Vue/React (Archivos estáticos compilados)
```

### 3.3. Configuración de Red (Docker)
Todos los contenedores se comunican a través de una red bridge definida por usuario llamada `meetlines-agent`.
*   **DNS Interno**: Los contenedores pueden resolverse entre sí por su `container_name`.
*   **Reverse Proxy**: Nginx actúa como puerta de enlace única, redirigiendo subdominios a puertos internos.

### 3.4. Mapa de Dominios (DNS)

| Dominio | Servicio | Puerto Interno | Contenedor |
| :--- | :--- | :--- | :--- |
| `app.meet-lines.com` | **Frontend** | Estático | Nginx (Host) |
| `docs.meet-lines.com` | **Wiki** | 8080 | `wikijs-app` |
| `n8n.meet-lines.com` | **N8N** | 5678 | `n8n` |
| `grafana.meet-lines.com` | **Grafana** | 3000 | `grafana` |
| `portainer.meet-lines.com` | **Portainer** | 9000 | `portainer` |

---

## 4. Catálogo de Servicios

### 4.1. Core Stack
*   **Docker**: Motor de contenedores.
*   **Docker Compose**: Orquestación de servicios.
*   **Nginx**: Reverse Proxy y Terminación SSL (Certbot).

### 4.2. Bases de Datos (`/projects/databases`)
*   **PostgreSQL 15**: Base de datos relacional principal.
    *   Puerto: `5432` (Solo localhost/Docker).
    *   Volumen: `./postgres_data`.
*   **Redis 6**: Caché y cola de mensajes.
    *   Puerto: `6379`.

### 4.3. Monitoreo (`/projects/monitoring`)
*   **Prometheus**: Recolección de métricas.
*   **Node Exporter**: Métricas del host (CPU, RAM).
*   **Grafana**: Visualización de dashboards.
*   **Portainer**: Interfaz visual para gestión de Docker.

### 4.4. Aplicaciones
*   **Backend**: .NET 8 API. Expone puerto `3001` al host para Nginx.
*   **Frontend**: SPA servida directamente por Nginx desde `/var/www/MeetLines-Frontend/dist`.

---

## 5. Ciclo de Vida y Despliegue (CI/CD)

Los despliegues están automatizados mediante **GitHub Actions**.

### 5.1. Backend (`deploy-backend.yml`)
*   **Trigger**: Push a rama `main`.
*   **Proceso**:
    1.  Runner conecta vía SSH al VPS.
    2.  Navega a `/var/www/MeetLines-Backend`.
    3.  `git fetch` + `git reset --hard origin/main`.
    4.  `docker compose up -d --build`.
    5.  Baja tiempo de inactividad (Zero-downtime no garantizado sin Swarm/K8s).

### 5.2. Frontend (`deploy-frontend.yml`)
*   **Trigger**: Push a rama `main`.
*   **Proceso**:
    1.  Runner compila el proyecto (`npm run build`).
    2.  Limpia `/var/www/MeetLines-Frontend/dist`.
    3.  Copia los nuevos archivos vía SCP.
    4.  Recarga Nginx (`systemctl reload nginx`).

---

## 6. Mantenimiento y Operaciones

### Comandos Útiles

**Verificar Estado de Servicios**
```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

**Verificar Logs en Tiempo Real**
```bash
# Backend
docker logs -f meetlines-backend

# Nginx (Errores de acceso web)
sudo tail -f /var/log/nginx/error.log
```

**Reiniciar Servicios de Infraestructura**
```bash
# Ejemplo: Reiniciar N8N
cd ~/projects/automation
docker compose restart
```

### Limitaciones Conocidas
*   **Escalabilidad Horizontal**: No implementada (Docker Standalone).
*   **Memoria**: Con 4GB, es crítico mantener configurados los `limits` de memoria en cada `docker-compose.yml` para evitar OOM Kills.
*   **Backups**: Configurados localmente en `~/projects/monitoring/backups` (Validar política de retención y off-site).

---

## 7. Plan de Mejoras (Roadmap)
*   [x] Automatización de despliegues (CI/CD).
*   [x] Centralización de logs y métricas.
*   [x] Hardening de SSH.
*   [ ] Implementación de backups automáticos a S3/Cloud.
*   [ ] Alertas de caída de servicio (vía Grafana/Alertmanager).
