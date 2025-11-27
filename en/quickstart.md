# Quickstart Guide

This Quickstart will help you get the MeetLines project running locally for development and testing.

Prerequisites:
- Git
- .NET SDK (if running the API)
- Docker and Docker Compose (for running services locally)

Steps:
1. Clone the repository:

```bash
git clone <repo-url>
cd MeetLines
```

2. Restore dependencies (for .NET projects):

```bash
dotnet restore
```

3. Start services with Docker Compose (if applicable):

```bash
docker compose up -d
```

4. Start the API (if running locally):

```bash
cd MeetLines.API
dotnet run
```

Visit `https://localhost:5001/swagger` to explore the API endpoints.

