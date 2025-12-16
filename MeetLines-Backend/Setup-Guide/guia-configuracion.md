# Guía de Configuración Local - MeetLines Backend

## Requisitos Previos

Antes de comenzar, asegúrate de tener instalado lo siguiente:

| Requisito | Versión | Propósito |
|-----------|---------|----------|
| .NET SDK | 8.0+ | Runtime y herramientas |
| PostgreSQL | 15+ | Base de datos |
| Docker | 20.10+ | Containerización |
| Docker Compose | 2.0+ | Orquestación de contenedores |
| Git | 2.0+ | Control de versiones |
| Visual Studio Code o Visual Studio | 2022+ | Editor de código (opcional) |

## Verificación de Requisitos

### Windows PowerShell
```powershell
# Verificar .NET SDK
dotnet --version

# Verificar Docker
docker --version
docker compose version

# Verificar Git
git --version
```

### Linux/macOS
```bash
# Verificar .NET SDK
dotnet --version

# Verificar Docker
docker --version
docker compose version

# Verificar Git
git --version
```

---

## Instalación de .NET 8 SDK

### Windows

1. Descargar desde [dotnet.microsoft.com](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
2. Ejecutar el instalador `.exe`
3. Seguir el asistente de instalación
4. Reiniciar la máquina

### Linux (Ubuntu/Debian)

```bash
# Descargar el repositorio
wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh
chmod +x dotnet-install.sh

# Instalar
./dotnet-install.sh --version latest

# Agregar al PATH
export DOTNET_ROOT=$(pwd)/.dotnet
export PATH=$DOTNET_ROOT:$PATH

# Verificar
dotnet --version
```

### macOS

```bash
# Con Homebrew
brew install dotnet

# Verificar
dotnet --version
```

---

## Instalación de Docker y Docker Compose

### Windows

1. Descargar [Docker Desktop para Windows](https://www.docker.com/products/docker-desktop)
2. Ejecutar el instalador
3. Reiniciar la máquina
4. Abrir PowerShell y verificar:

```powershell
docker --version
docker compose version
```

### Linux

```bash
# Actualizar paquetes
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Agregar repositorio de Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Instalar Docker
sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo apt install -y docker-compose-plugin

# Verificar
docker --version
docker compose version
```

### macOS

```bash
# Con Homebrew
brew install --cask docker

# Iniciar Docker Desktop
open /Applications/Docker.app

# Verificar
docker --version
docker compose version
```

---

## Clonar el Repositorio

```bash
# Clonar el repositorio
git clone https://github.com/Gula-Riwi/MeetLines-Backend.git
cd MeetLines-Backend

# Verificar que estás en la rama correcta
git status
```

---

## Configuración de Variables de Entorno

### 1. Crear archivo `.env`

En la raíz del proyecto, crea un archivo `.env`:

```bash
# Copiar ejemplo (si existe)
cp .env.example .env
```

### 2. Configurar variables

Edita el archivo `.env` con tus valores:

```env
# PostgreSQL
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=meetlines_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_secure_password
POSTGRES_CONNECTION_STRING=Server=localhost;Port=5432;Database=meetlines_db;User Id=postgres;Password=your_secure_password;

# ASP.NET Core
ASPNETCORE_ENVIRONMENT=Development
ASPNETCORE_URLS=https://localhost:5001
JWT_KEY=your_jwt_secret_key_min_256_bits_long
JWT_ISSUER=meetlines
JWT_AUDIENCE=meetlines-api

# OAuth Providers
DISCORD_CLIENT_ID=your_discord_client_id
DISCORD_CLIENT_SECRET=your_discord_client_secret
DISCORD_REDIRECT_URI=https://localhost:5001/api/auth/oauth/discord/callback

FACEBOOK_APP_ID=your_facebook_app_id
FACEBOOK_APP_SECRET=your_facebook_app_secret
FACEBOOK_REDIRECT_URI=https://localhost:5001/api/auth/oauth/facebook/callback

GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_REDIRECT_URI=https://localhost:5001/api/auth/oauth/google/callback

# Servicios Externos
WHATSAPP_API_URL=https://graph.instagram.com/v18.0
WHATSAPP_API_TOKEN=your_whatsapp_token
WHATSAPP_BUSINESS_ACCOUNT_ID=your_business_account_id

TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TWILIO_PHONE_NUMBER=your_twilio_phone

SENDGRID_API_KEY=your_sendgrid_api_key
SENDGRID_FROM_EMAIL=noreply@meet-lines.com

# Configuración de la Aplicación
CORS_ORIGINS=https://localhost:3000,https://localhost:5173
LOG_LEVEL=Information
DATABASE_MIGRATION_ON_STARTUP=true
```

---

## Levantar la Infraestructura con Docker Compose

### 1. Iniciar PostgreSQL

```bash
# Desde la raíz del proyecto
docker compose up -d

# Verificar que el contenedor está corriendo
docker ps

# Ver logs
docker compose logs -f postgres
```

### 2. Verificar conexión a PostgreSQL

```bash
# Conectarse a PostgreSQL
docker compose exec postgres psql -U postgres -d meetlines_db

# Ver tablas (dentro de psql)
\dt

# Salir
\q
```

---

## Restaurar dependencias e inicializar la BD

### 1. Restaurar paquetes NuGet

```bash
# Desde la raíz del proyecto
dotnet restore
```

### 2. Aplicar migraciones

```bash
# Navegar a la capa de API
cd MeetLines.API

# Aplicar migraciones
dotnet ef database update

# Ver migraciones aplicadas
dotnet ef migrations list
```

Si ocurren errores, verifica:
- La conexión a PostgreSQL
- Las variables de entorno `.env`
- La ruta de ejecución

---

## Ejecutar la Aplicación

### Opción 1: Ejecutar desde Visual Studio

1. Abre `MeetLines.sln` en Visual Studio
2. Establece `MeetLines.API` como proyecto de inicio
3. Presiona `F5` o haz clic en "Ejecutar"
4. La aplicación se abrirá en `https://localhost:5001`

### Opción 2: Ejecutar desde CLI

```bash
# Desde la carpeta MeetLines.API
cd MeetLines.API
dotnet run

# O especificar el ambiente
dotnet run --configuration Development
```

### Opción 3: Usar Docker Compose (completo)

```bash
# Desde la raíz del proyecto
docker compose up -d

# Ver logs de la API
docker compose logs -f api
```

---

## Verificar que la API está funcionando

### 1. Acceder a Swagger

Abre tu navegador en: `https://localhost:5001/swagger`

Deberías ver la documentación interactiva de la API.

### 2. Health Check

```bash
curl https://localhost:5001/api/health/health \
  -H "Accept: application/json"
```

**Respuesta esperada:**
```json
{
  "status": "healthy",
  "timestamp": "2025-01-20T14:30:00Z"
}
```

### 3. Registrar un usuario

```bash
curl -X POST https://localhost:5001/api/Auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "TestPassword123!",
    "firstName": "Test",
    "lastName": "User"
  }'
```

---

## Estructura del Proyecto Después de la Restauración

```
MeetLines-Backend/
├── MeetLines.API/              # Capa de API
│   ├── Controllers/
│   ├── Filters/
│   ├── Middleware/
│   ├── Program.cs
│   ├── appsettings.json
│   └── appsettings.Development.json
│
├── MeetLines.Application/      # Capa de Aplicación
│   ├── UseCases/
│   ├── Services/
│   ├── DTOs/
│   ├── Validators/
│   └── Interfaces/
│
├── MeetLines.Domain/           # Capa de Dominio
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Repositories/
│   └── Enums/
│
├── MeetLines.Infrastructure/   # Capa de Infraestructura
│   ├── Data/
│   ├── Repositories/
│   ├── Services/
│   └── IoC/
│
├── MeetLines.Tests/            # Pruebas Unitarias
│   └── Services/
│
├── SharedKernel/               # Utilidades Compartidas
│   └── Utilities/
│
├── docker-compose.yaml         # Composición de contenedores
├── pom.xml                     # (Antiguo) Configuración Maven
├── .env.example                # Ejemplo de variables
└── README.md
```

---

## Solución de Problemas Comunes

### Error: "Unable to connect to PostgreSQL"

**Solución:**
1. Verifica que Docker está corriendo: `docker ps`
2. Verifica la conexión: `docker compose logs postgres`
3. Recrea el contenedor: `docker compose down && docker compose up -d postgres`

### Error: "Migration failed"

**Solución:**
1. Elimina la base de datos: `docker compose exec postgres dropdb -U postgres meetlines_db`
2. Recreala: `docker compose exec postgres createdb -U postgres meetlines_db`
3. Reaplica migraciones: `dotnet ef database update`

### Error: "Port 5432 already in use"

**Solución:**
```bash
# Cambiar puerto en docker-compose.yaml
# Modificar: 5432:5432 a 5433:5432
# Actualizar CONNECTION_STRING en .env

# O liberar el puerto
docker compose down
```

### Error: "SSL/Certificate issues"

**Solución (solo desarrollo):**
```bash
# Ignorar certificado SSL en desarrollo
export ASPNETCORE_ENVIRONMENT=Development

# O en .env
ASPNETCORE_ENVIRONMENT=Development
```

### Error: "API en HTTPS pero navegador dice inseguro"

**Solución:**
1. Es normal en desarrollo
2. Haz clic en "Avanzado" → "Continuar"
3. O accede a `http://localhost:5000` (sin SSL)

---

## Comandos Útiles de Desarrollo

### Entity Framework

```bash
# Crear migración
dotnet ef migrations add "MigrationName" -p MeetLines.Infrastructure

# Aplicar migraciones
dotnet ef database update

# Ver migraciones
dotnet ef migrations list

# Revertir última migración
dotnet ef migrations remove
```

### Docker Compose

```bash
# Iniciar servicios
docker compose up -d

# Detener servicios
docker compose down

# Ver logs
docker compose logs -f [service_name]

# Reconstruir imágenes
docker compose build

# Eliminar todo (contenedores, volúmenes)
docker compose down -v
```

### Ejecución de Pruebas

```bash
# Ejecutar todas las pruebas
dotnet test

# Pruebas solo de un proyecto
dotnet test MeetLines.Tests

# Con cobertura
dotnet test /p:CollectCoverageOnTestExecution=true
```

---

## Referencias y Recursos

- [.NET Documentation](https://learn.microsoft.com/en-us/dotnet/)
- [Entity Framework Core](https://learn.microsoft.com/en-us/ef/core/)
- [ASP.NET Core Best Practices](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/best-practices)
- [Docker Documentation](https://docs.docker.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

---

## Próximos Pasos

1. ✅ Completaste la configuración local
2. 📖 Lee la documentación de arquitectura
3. 🔑 Configura OAuth en Discord/Facebook/Google
4. 📝 Revisa los endpoints en Swagger
5. 🧪 Ejecuta las pruebas
6. 🚀 Comienza a desarrollar

¡Bienvenido al equipo de MeetLines Backend!
