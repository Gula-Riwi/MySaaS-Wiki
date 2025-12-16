# Frontend Local Setup Guide - Meetlines

## Prerequisites

Before you start, ensure you have the following installed:

| Requirement | Version | Purpose |
|-------------|---------|---------|
| Node.js | 18.0+ | JavaScript runtime |
| npm | 9.0+ | Package manager |
| Git | 2.0+ | Version control |
| Visual Studio Code | Latest | Code editor (recommended) |

## Verify Prerequisites

```bash
# Check Node.js
node --version

# Check npm
npm --version

# Check Git
git --version
```

## Clone the Repository

```bash
# Clone the repository
git clone https://github.com/Gula-Riwi/Meetlines-Frontend.git
cd Meetlines-Frontend

# Verify branch
git status
```

## Set Up Environment Variables

### 1. Create `.env` file

```bash
# Copy example
cp .env.example .env
```

### 2. Configure variables

Edit the `.env` file:

```env
# API Configuration
VITE_API_URL=https://services.meet-lines.com/api
VITE_API_TIMEOUT=30000

# Authentication
VITE_JWT_STORAGE_KEY=meetlines_token
VITE_REFRESH_TOKEN_STORAGE_KEY=meetlines_refresh_token

# OAuth Credentials (for development)
VITE_DISCORD_CLIENT_ID=your_discord_client_id
VITE_FACEBOOK_APP_ID=your_facebook_app_id
VITE_GOOGLE_CLIENT_ID=your_google_client_id

# Application
VITE_APP_NAME=MeetLines
VITE_APP_VERSION=1.0.0
VITE_ENVIRONMENT=development

# WebSocket (for real-time chat)
VITE_WS_URL=wss://services.meet-lines.com/ws
```

## Install Dependencies

```bash
# Install npm packages
npm install

# Or with yarn
yarn install

# Or with pnpm
pnpm install
```

## Run Development Server

```bash
# Start development server with HMR
npm run dev

# The app will be available at:
# http://localhost:5173 (or another available port)
```

## Build for Production

```bash
# Build optimized version
npm run build

# Preview production build
npm run preview
```

## Project Structure

```
Meetlines-Frontend/
├── public/                      # Static files
├── src/
│   ├── components/              # Reusable Vue components
│   │   ├── common/              # Common components
│   │   ├── forms/               # Form components
│   │   ├── cards/               # Card components
│   │   └── modals/              # Modal components
│   ├── views/                   # Page components
│   │   ├── Dashboard.vue
│   │   ├── Projects.vue
│   │   ├── Appointments.vue
│   │   └── ...
│   ├── services/                # API services
│   │   ├── api.js
│   │   ├── authService.js
│   │   └── ...
│   ├── stores/                  # Pinia stores
│   │   ├── authStore.js
│   │   ├── projectStore.js
│   │   └── ...
│   ├── router/                  # Vue Router config
│   │   └── index.js
│   ├── utils/                   # Utility functions
│   ├── assets/                  # Images, fonts, etc.
│   ├── App.vue                  # Root component
│   ├── main.js                  # Entry point
│   └── styles.css               # Global styles
├── index.html
├── vite.config.js              # Vite configuration
├── tailwind.config.js          # Tailwind CSS config
├── .env.example                # Example environment vars
├── package.json
└── README.md
```

## Available Scripts

```bash
# Development server
npm run dev

# Production build
npm run build

# Preview production build
npm run preview

# Run linter
npm run lint

# Format code
npm run format

# Run tests (if configured)
npm run test
```

## Develop with HMR (Hot Module Replacement)

The development server automatically reloads your browser when you make changes:

1. Start the dev server: `npm run dev`
2. Open `http://localhost:5173`
3. Edit any `.vue` file and see changes instantly

## Debugging

### Browser DevTools

1. Open Chrome DevTools (`F12`)
2. Go to `Sources` tab
3. Set breakpoints in your code
4. Reload the page

### Vue DevTools

Install the [Vue DevTools browser extension](https://devtools.vuejs.org/) to inspect:
- Component hierarchy
- Props and emits
- Store state
- Router history

## Common Issues

### "Port 5173 already in use"

```bash
# Kill the process using port 5173
# Linux/macOS
lsof -ti:5173 | xargs kill -9

# Windows
netstat -ano | findstr :5173
taskkill /PID <PID> /F
```

### "Module not found" errors

```bash
# Clear node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

### "API calls return 401 Unauthorized"

Check that:
1. `.env` file has correct `VITE_API_URL`
2. You're logged in (token in localStorage)
3. Backend is running and accessible

### "Tailwind CSS not applying styles"

```bash
# Make sure Tailwind is building correctly
npm run build

# If issue persists, rebuild CSS
rm -rf node_modules/.vite
npm run dev
```

## Contributing

### Code Standards

- Use Vue 3 Composition API with `<script setup>`
- Follow kebab-case for component names
- Use camelCase for JavaScript variables
- Use Tailwind CSS classes instead of custom CSS when possible

### Component Example

```vue
<script setup>
import { ref, computed } from 'vue';

const message = ref('Hello');
const doubled = computed(() => message.value.length * 2);

const updateMessage = (newValue) => {
  message.value = newValue;
};
</script>

<template>
  <div>
    <input
      v-model="message"
      type="text"
      placeholder="Type something..."
    />
    <p>Length: {{ doubled }}</p>
  </div>
</template>

<style scoped>
/* Use Tailwind utilities in template instead of writing CSS here */
</style>
```

## Performance Tips

1. **Lazy load routes:**
   ```javascript
   component: () => import('@/views/Dashboard.vue')
   ```

2. **Use `v-if` instead of `v-show` for heavy components**

3. **Optimize images with `loading="lazy"`**

4. **Use Pinia stores for shared state instead of prop drilling**

## Deployment

### Build for production

```bash
npm run build
```

The optimized files will be in the `dist/` folder.

### Docker deployment

```bash
# Build Docker image
docker build -t meetlines-frontend .

# Run container
docker run -p 80:80 meetlines-frontend
```

## Resources

- [Vue 3 Documentation](https://vuejs.org/)
- [Vite Documentation](https://vitejs.dev/)
- [Tailwind CSS Documentation](https://tailwindcss.com/)
- [Pinia Documentation](https://pinia.vuejs.org/)
- [Vue Router Documentation](https://router.vuejs.org/)

## Next Steps

1. ✅ Completed local setup
2. 📖 Read the technical architecture documentation
3. 🚀 Start the dev server: `npm run dev`
4. 🔑 Configure authentication
5. 🧪 Run tests
6. 💻 Start building features

Welcome to the MeetLines Frontend team!
