# Setup Guide - Meetlines Frontend

## Prerequisites

Ensure you have:

| Requirement | Version | Purpose |
|-------------|---------|---------|
| Node.js | 18.0+ | JavaScript runtime |
| npm | 9.0+ | Package manager |
| Git | 2.0+ | Version control |
| VS Code | Latest | Code editor |

## Verify Requirements

```bash
node --version
npm --version
git --version
```

## Clone Repository

```bash
git clone https://github.com/Gula-Riwi/Meetlines-Frontend.git
cd Meetlines-Frontend
git status
```

## Environment Setup

### Create `.env` file

```bash
cp .env.example .env
```

### Configure variables

```env
# API Configuration
VITE_API_URL=https://services.meet-lines.com/api
VITE_API_TIMEOUT=30000

# Authentication
VITE_JWT_STORAGE_KEY=meetlines_token
VITE_REFRESH_TOKEN_STORAGE_KEY=meetlines_refresh_token

# OAuth Credentials
VITE_DISCORD_CLIENT_ID=your_discord_client_id
VITE_FACEBOOK_APP_ID=your_facebook_app_id
VITE_GOOGLE_CLIENT_ID=your_google_client_id

# Application
VITE_APP_NAME=MeetLines
VITE_ENVIRONMENT=development

# WebSocket
VITE_WS_URL=wss://services.meet-lines.com/ws
```

## Install Dependencies

```bash
npm install
# or: yarn install
# or: pnpm install
```

## Run Development Server

```bash
npm run dev
# Opens at http://localhost:5173
```

## Build for Production

```bash
npm run build
npm run preview
```

## Available Commands

```bash
npm run dev       # Development server
npm run build     # Production build
npm run preview   # Preview build
npm run lint      # Run linter
npm run format    # Format code
npm run test      # Run tests
```

## Troubleshooting

### Port 5173 already in use

```bash
# Linux/macOS
lsof -ti:5173 | xargs kill -9

# Windows
netstat -ano | findstr :5173
taskkill /PID <PID> /F
```

### Module not found

```bash
rm -rf node_modules package-lock.json
npm install
```

### API returns 401

Check:
1. `.env` has correct `VITE_API_URL`
2. You're logged in (token in localStorage)
3. Backend is running

## Debugging

### Vue DevTools

Install [Vue DevTools extension](https://devtools.vuejs.org/) to inspect:
- Component hierarchy
- Props and emits
- Store state
- Router history

### Browser DevTools

1. Press `F12`
2. Go to `Sources` tab
3. Set breakpoints
4. Reload page

## Next Steps

1. ✅ Setup completed
2. 🚀 Run: `npm run dev`
3. 📖 Read architecture documentation
4. 🔑 Configure authentication
5. 💻 Start building features

Welcome to MeetLines Frontend!
