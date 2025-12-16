---
title: Infrastructure and VPS
description: Complete technical documentation of the production environment.
published: true
tags: infrastructure, vps, docker, security
editor: markdown
---

# Infrastructure and VPS

Technical documentation for the production server ("Gula-VPS"). This document serves as the single source of truth for system administration, deployment, and maintenance.

---

## 1. Server Specifications

| Resource | Detail |
| :--- | :--- |
| **Operating System** | Ubuntu 22.04 LTS (Jammy Jellyfish) |
| **CPU** | 2 vCPU |
| **RAM** | 4 GB |
| **Storage** | 40 GB SSD |
| **Swap** | 2 GB (Configured in `/swapfile`) |
| **Public IP** | 46.224.13.157 |

### 1.1. Users and Access
*   **Root Access**: Disabled via SSH.
*   **Authentication**: Exclusively via **SSH Keys** (passwords disabled).
*   **Administrator Users**:
    *   `miguel_arias` 
    *   `omar_andres` 
    *   `jhon_rojas` 
    *   `emmanuel_rendon` 
    *   `jhonatan_morales` 
    *   `juan_sepulveda`
      

---

## 2. Access and Security (Hardening)

### 2.1. Intrusion Protection (Fail2Ban)
**Fail2Ban** is used to protect the SSH port and Nginx against brute force attacks.
*   **SSH Jail**: Bans the IP after 5 failed attempts. Ban time: 1 hour.

### 2.2. SSH Key Management
To grant access to a new developer:
1.  Request their public key (`id_rsa.pub`).
2.  Log in as administrator.
3.  Add the key:
    ```bash
    # In the corresponding user
    echo "ssh-rsa AAAA..." >> ~/.ssh/authorized_keys
    ```

---

## 3. Folder and Network Structure

The infrastructure is divided into two main locations according to their purpose:

### 3.1. Infrastructure Services (`~/projects`)
Located in `/home/miguel_arias/projects` (or `/root/projects` historically). Contains support and automation services.

```
~/projects/
├── automation/       # N8N (Flow automation)
├── databases/        # Central PostgreSQL and Redis
├── monitoring/       # Grafana, Prometheus, Portainer
└── wiki/             # Wiki.js (Documentation)
```

### 3.2. Business Applications (`/var/www`)
Standard location for automated web deployments.

```
/var/www/
├── MeetLines-Backend/   # .NET Core API (Deployed via Git Pull)
└── MeetLines-Frontend/  # Vue/React Frontend (Compiled static files)
```

### 3.3. Network Configuration (Docker)
All containers communicate through a user-defined bridge network called `meetlines-agent`.
*   **Internal DNS**: Containers can resolve each other by their `container_name`.
*   **Reverse Proxy**: Nginx acts as the single gateway, redirecting subdomains to internal ports.

### 3.4. Domain Map (DNS)

| Domain | Service | Internal Port | Container |
| :--- | :--- | :--- | :--- |
| `app.meet-lines.com` | **Frontend** | Static | Nginx (Host) |
| `docs.meet-lines.com` | **Wiki** | 8080 | `wikijs-app` |
| `n8n.meet-lines.com` | **N8N** | 5678 | `n8n` |
| `grafana.meet-lines.com` | **Grafana** | 3000 | `grafana` |
| `portainer.meet-lines.com` | **Portainer** | 9000 | `portainer` |

---

## 4. Service Catalog

### 4.1. Core Stack
*   **Docker**: Container engine.
*   **Docker Compose**: Service orchestration.
*   **Nginx**: Reverse Proxy and SSL Termination (Certbot).

### 4.2. Databases (`/projects/databases`)
*   **PostgreSQL 15**: Main relational database.
    *   Port: `5432` (Localhost/Docker only).
    *   Volume: `./postgres_data`.
*   **Redis 6**: Cache and message queue.
    *   Port: `6379`.

### 4.3. Monitoring (`/projects/monitoring`)
*   **Prometheus**: Metrics collection.
*   **Node Exporter**: Host metrics (CPU, RAM).
*   **Grafana**: Dashboard visualization.
*   **Portainer**: Visual interface for Docker management.

### 4.4. Applications
*   **Backend**: .NET 8 API. Exposes port `3001` to the host for Nginx.
*   **Frontend**: SPA served directly by Nginx from `/var/www/MeetLines-Frontend/dist`.

---

## 5. Lifecycle and Deployment (CI/CD)

Deployments are automated using **GitHub Actions**.

### 5.1. Backend (`deploy-backend.yml`)
*   **Trigger**: Push to `main` branch.
*   **Process**:
    1.  Runner connects via SSH to the VPS.
    2.  Navigates to `/var/www/MeetLines-Backend`.
    3.  `git fetch` + `git reset --hard origin/main`.
    4.  `docker compose up -d --build`.
    5.  Low downtime (Zero-downtime not guaranteed without Swarm/K8s).

### 5.2. Frontend (`deploy-frontend.yml`)
*   **Trigger**: Push to `main` branch.
*   **Process**:
    1.  Runner builds the project (`npm run build`).
    2.  Cleans `/var/www/MeetLines-Frontend/dist`.
    3.  Copies new files via SCP.
    4.  Reloads Nginx (`systemctl reload nginx`).

---

## 6. Maintenance and Operations

### Useful Commands

**Check Service Status**
```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

**Check Logs in Real Time**
```bash
# Backend
docker logs -f meetlines-backend

# Nginx (Web access errors)
sudo tail -f /var/log/nginx/error.log
```

**Restart Infrastructure Services**
```bash
# Example: Restart N8N
cd ~/projects/automation
docker compose restart
```

### Known Limitations
*   **Horizontal Scalability**: Not implemented (Docker Standalone).
*   **Memory**: With 4GB, it is critical to keep memory `limits` configured in each `docker-compose.yml` to avoid OOM Kills.
*   **Backups**: Configured locally in `~/projects/monitoring/backups` (Validate retention and off-site policy).

---

## 7. Improvement Plan (Roadmap)
*   [x] Deployment Automation (CI/CD).
*   [x] Centralization of logs and metrics.
*   [x] SSH Hardening.
*   [ ] Implementation of automatic backups to S3/Cloud.
*   [ ] Service downtime alerts (via Grafana/Alertmanager).
