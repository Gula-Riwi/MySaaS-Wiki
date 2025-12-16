# Local Setup Guide - MeetLines Backend

## Prerequisites

Before starting, make sure you have the following installed:

| Requirement | Version | Purpose |
|-------------|---------|---------|
| .NET SDK | 8.0+ | Runtime and tools |
| PostgreSQL | 15+ | Database |
| Docker | 20.10+ | Containerization |
| Docker Compose | 2.0+ | Container orchestration |
| Git | 2.0+ | Version control |
| Visual Studio Code or Visual Studio | 2022+ | Code editor (optional) |

## Verify Requirements

### Windows PowerShell
```powershell
# Verify .NET SDK
dotnet --version

# Verify Docker
docker --version
docker compose version

# Verify Git
git --version
```

### Linux/macOS
```bash
# Verify .NET SDK
dotnet --version

# Verify Docker
docker --version
docker compose version

# Verify Git
git --version
```

---

## Install .NET 8 SDK

### Windows

1. Download from [dotnet.microsoft.com](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
2. Run the `.exe` installer
3. Follow the installation wizard
4. Restart your machine

### Linux (Ubuntu/Debian)

```bash
# Download the installer
wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh
chmod +x dotnet-install.sh

# Install
./dotnet-install.sh --version latest

# Add to PATH
export DOTNET_ROOT=$(pwd)/.dotnet
export PATH=$DOTNET_ROOT:$PATH

# Verify
dotnet --version
```

### macOS

```bash
# With Homebrew
brew install dotnet

# Verify
dotnet --version
```

---

## Install Docker and Docker Compose

### Windows

1. Download [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop)
2. Run the installer
3. Restart your machine
4. Open PowerShell and verify:

```powershell
docker --version
docker compose version
```

### Linux

```bash
# Update packages
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker repository
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Install Docker
sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo apt install -y docker-compose-plugin

# Verify
docker --version
docker compose version
```

### macOS

```bash
# With Homebrew
brew install --cask docker

# Start Docker Desktop
open /Applications/Docker.app

# Verify
docker --version
docker compose version
```

---

## Clone the Repository

```bash
# Clone the repository
git clone https://github.com/Gula-Riwi/MeetLines-Backend.git
cd MeetLines-Backend

# Check your current branch
git status
```

---

## Set Up Environment Variables

### 1. Create `.env` file

In the project root, create an `.env` file:

```bash
# Copy example (if it exists)
cp .env.example .env
```

### 2. Configure variables

Edit the `.env` file with your values:

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

# External Services
WHATSAPP_API_URL=https://graph.instagram.com/v18.0
WHATSAPP_API_TOKEN=your_whatsapp_token
WHATSAPP_BUSINESS_ACCOUNT_ID=your_business_account_id

TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TWILIO_PHONE_NUMBER=your_twilio_phone

SENDGRID_API_KEY=your_sendgrid_api_key
SENDGRID_FROM_EMAIL=noreply@meet-lines.com

# Application Settings
CORS_ORIGINS=https://localhost:3000,https://localhost:5173
LOG_LEVEL=Information
DATABASE_MIGRATION_ON_STARTUP=true
```

---

## Start Infrastructure with Docker Compose

### 1. Start PostgreSQL

```bash
# From the project root
docker compose up -d

# Verify the container is running
docker ps

# View logs
docker compose logs -f postgres
```

### 2. Verify PostgreSQL connection

```bash
# Connect to PostgreSQL
docker compose exec postgres psql -U postgres -d meetlines_db

# View tables (inside psql)
\dt

# Exit
\q
```

---

## Restore dependencies and initialize DB

### 1. Restore NuGet packages

```bash
# From the project root
dotnet restore
```

### 2. Apply migrations

```bash
# Navigate to the API layer
cd MeetLines.API

# Apply migrations
dotnet ef database update

# View applied migrations
dotnet ef migrations list
```

If errors occur, verify:
- PostgreSQL connection
- Environment variables in `.env`
- Execution path

---

## Run the Application

### Option 1: Run from Visual Studio

1. Open `MeetLines.sln` in Visual Studio
2. Set `MeetLines.API` as the startup project
3. Press `F5` or click "Run"
4. The application will open at `https://localhost:5001`

### Option 2: Run from CLI

```bash
# From the MeetLines.API folder
cd MeetLines.API
dotnet run

# Or specify environment
dotnet run --configuration Development
```

### Option 3: Use Docker Compose (full stack)

```bash
# From the project root
docker compose up -d

# View API logs
docker compose logs -f api
```

---

## Verify the API is working

### 1. Access Swagger

Open your browser at: `https://localhost:5001/swagger`

You should see the interactive API documentation.

### 2. Health Check

```bash
curl https://localhost:5001/api/health/health \
  -H "Accept: application/json"
```

**Expected response:**
```json
{
  "status": "healthy",
  "timestamp": "2025-01-20T14:30:00Z"
}
```

### 3. Register a user

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

## Project Structure After Restoration

```
MeetLines-Backend/
├── MeetLines.API/              # API Layer
│   ├── Controllers/
│   ├── Filters/
│   ├── Middleware/
│   ├── Program.cs
│   ├── appsettings.json
│   └── appsettings.Development.json
│
├── MeetLines.Application/      # Application Layer
│   ├── UseCases/
│   ├── Services/
│   ├── DTOs/
│   ├── Validators/
│   └── Interfaces/
│
├── MeetLines.Domain/           # Domain Layer
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Repositories/
│   └── Enums/
│
├── MeetLines.Infrastructure/   # Infrastructure Layer
│   ├── Data/
│   ├── Repositories/
│   ├── Services/
│   └── IoC/
│
├── MeetLines.Tests/            # Unit Tests
│   └── Services/
│
├── SharedKernel/               # Shared Utilities
│   └── Utilities/
│
├── docker-compose.yaml         # Container composition
├── pom.xml                     # (Legacy) Maven configuration
├── .env.example                # Environment variables example
└── README.md
```

---

## Troubleshooting Common Issues

### Error: "Unable to connect to PostgreSQL"

**Solution:**
1. Verify Docker is running: `docker ps`
2. Check the connection: `docker compose logs postgres`
3. Recreate the container: `docker compose down && docker compose up -d postgres`

### Error: "Migration failed"

**Solution:**
1. Delete the database: `docker compose exec postgres dropdb -U postgres meetlines_db`
2. Recreate it: `docker compose exec postgres createdb -U postgres meetlines_db`
3. Reapply migrations: `dotnet ef database update`

### Error: "Port 5432 already in use"

**Solution:**
```bash
# Change port in docker-compose.yaml
# Modify: 5432:5432 to 5433:5432
# Update CONNECTION_STRING in .env

# Or free up the port
docker compose down
```

### Error: "SSL/Certificate issues"

**Solution (for development only):**
```bash
# Ignore SSL certificate in development
export ASPNETCORE_ENVIRONMENT=Development

# Or in .env
ASPNETCORE_ENVIRONMENT=Development
```

### Error: "API in HTTPS but browser says insecure"

**Solution:**
1. This is normal in development
2. Click "Advanced" → "Continue"
3. Or access `http://localhost:5000` (without SSL)

---

## Useful Development Commands

### Entity Framework

```bash
# Create migration
dotnet ef migrations add "MigrationName" -p MeetLines.Infrastructure

# Apply migrations
dotnet ef database update

# View migrations
dotnet ef migrations list

# Revert last migration
dotnet ef migrations remove
```

### Docker Compose

```bash
# Start services
docker compose up -d

# Stop services
docker compose down

# View logs
docker compose logs -f [service_name]

# Rebuild images
docker compose build

# Remove everything (containers, volumes)
docker compose down -v
```

### Run Tests

```bash
# Run all tests
dotnet test

# Run tests from specific project
dotnet test MeetLines.Tests

# With coverage
dotnet test /p:CollectCoverageOnTestExecution=true
```

---

## References and Resources

- [.NET Documentation](https://learn.microsoft.com/en-us/dotnet/)
- [Entity Framework Core](https://learn.microsoft.com/en-us/ef/core/)
- [ASP.NET Core Best Practices](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/best-practices)
- [Docker Documentation](https://docs.docker.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

---

## Next Steps

1. ✅ You've completed local setup
2. 📖 Read the architecture documentation
3. 🔑 Configure OAuth on Discord/Facebook/Google
4. 📝 Review the endpoints in Swagger
5. 🧪 Run the tests
6. 🚀 Start developing

Welcome to the MeetLines Backend team!
