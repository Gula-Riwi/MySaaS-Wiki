# Technical Architecture - Meetlines Frontend

## Overview

Meetlines Frontend is a modern web application built with **Vue 3**, **Vite**, and **Tailwind CSS**. It provides a responsive user interface for businesses to manage their projects, services, appointments, and real-time conversations.

## Technology Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| Vue.js | 3.x | Reactive JavaScript framework |
| Vite | 5.x+ | Build tool and dev server |
| Tailwind CSS | 3.x | Utility-first CSS framework |
| JavaScript ES6+ | - | Programming language |
| Axios | 1.x | HTTP client |
| Vue Router | 4.x | SPA routing |
| Pinia | 2.x | State management |

## Project Structure

```
Meetlines-Frontend/
├── public/                      # Static files
│   └── favicon.ico
├── src/
│   ├── assets/                  # Images, icons, fonts
│   │   └── images/
│   ├── components/              # Reusable components
│   │   ├── common/              # Generic components
│   │   │   ├── Header.vue
│   │   │   ├── Sidebar.vue
│   │   │   ├── Footer.vue
│   │   │   └── Navbar.vue
│   │   ├── forms/               # Form components
│   │   │   ├── AppointmentForm.vue
│   │   │   ├── ServiceForm.vue
│   │   │   └── LoginForm.vue
│   │   ├── cards/               # Card components
│   │   │   ├── ProjectCard.vue
│   │   │   ├── ServiceCard.vue
│   │   │   └── AppointmentCard.vue
│   │   └── modals/              # Modals and dialogs
│   │       ├── ConfirmModal.vue
│   │       └── EditModal.vue
│   ├── views/                   # Full page views
│   │   ├── Dashboard.vue        # Main dashboard
│   │   ├── Projects.vue         # Project management
│   │   ├── Appointments.vue     # Appointment management
│   │   ├── Services.vue         # Service management
│   │   ├── Conversations.vue    # Chat and conversations
│   │   ├── Feedback.vue         # Feedback
│   │   ├── Settings.vue         # Settings
│   │   └── Login.vue            # Authentication
│   ├── router/                  # Route configuration
│   │   └── index.js
│   ├── services/                # API services
│   │   ├── api.js               # Axios config
│   │   ├── authService.js       # Auth service
│   │   ├── projectService.js    # Projects service
│   │   ├── appointmentService.js # Appointments service
│   │   └── chatService.js       # Chat service
│   ├── stores/                  # Global state (Pinia)
│   │   ├── authStore.js         # Auth state
│   │   ├── projectStore.js      # Projects state
│   │   └── uiStore.js           # UI state
│   ├── utils/                   # Utilities and helpers
│   │   ├── validators.js        # Validators
│   │   ├── formatters.js        # Formatters
│   │   ├── constants.js         # Constants
│   │   └── helpers.js           # Helper functions
│   ├── App.vue                  # Root component
│   ├── main.js                  # Entry point
│   └── styles.css               # Global styles
├── .env.example                 # Environment variables example
├── index.html                   # Main HTML
├── package.json                 # Dependencies
├── vite.config.js               # Vite config
├── tailwind.config.js           # Tailwind CSS config
├── postcss.config.js            # PostCSS config
├── Dockerfile                   # Docker image
├── nginx.conf                   # Nginx config
└── README.md
```

## Component Architecture (Atomic Design)

### 1. Atoms (Basic components)

```vue
<!-- Button.vue -->
<template>
  <button
    :class="['btn', `btn-${variant}`, `btn-${size}`]"
    @click="$emit('click')"
  >
    <slot />
  </button>
</template>

<script setup>
defineProps({
  variant: { type: String, default: 'primary' },
  size: { type: String, default: 'md' }
});
</script>

<style scoped>
.btn {
  @apply px-4 py-2 rounded font-semibold transition;
}
.btn-primary {
  @apply bg-blue-500 text-white hover:bg-blue-600;
}
</style>
```

### 2. Molecules (Composite components)

```vue
<!-- SearchBar.vue -->
<template>
  <div class="search-bar">
    <input
      v-model="searchQuery"
      type="text"
      placeholder="Search..."
      @input="handleSearch"
    />
    <Button variant="secondary" @click="clearSearch">Clear</Button>
  </div>
</template>

<script setup>
import Button from '@/components/atoms/Button.vue';
import { ref } from 'vue';

const searchQuery = ref('');
const emit = defineEmits(['search']);

const handleSearch = () => {
  emit('search', searchQuery.value);
};

const clearSearch = () => {
  searchQuery.value = '';
  emit('search', '');
};
</script>
```

### 3. Organisms (Complex components)

```vue
<!-- AppointmentForm.vue -->
<template>
  <form @submit.prevent="submitForm">
    <div class="form-group">
      <label>Service</label>
      <ServiceSelector v-model="form.serviceId" />
    </div>

    <div class="form-group">
      <label>Customer</label>
      <CustomerInput v-model="form.customerPhone" />
    </div>

    <div class="form-group">
      <label>Date and Time</label>
      <DateTimePicker v-model="form.scheduledTime" />
    </div>

    <Button type="submit" :loading="isLoading">Create Appointment</Button>
  </form>
</template>
```

## State Management (Pinia)

### Auth Store

```javascript
// stores/authStore.js
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import * as authService from '@/services/authService';

export const useAuthStore = defineStore('auth', () => {
  const user = ref(null);
  const token = ref(localStorage.getItem('token'));
  const isLoading = ref(false);

  const isAuthenticated = computed(() => !!token.value);

  const login = async (email, password) => {
    isLoading.value = true;
    try {
      const response = await authService.login(email, password);
      token.value = response.accessToken;
      localStorage.setItem('token', response.accessToken);
      await fetchUser();
      return true;
    } catch (error) {
      console.error('Login failed:', error);
      return false;
    } finally {
      isLoading.value = false;
    }
  };

  const logout = () => {
    token.value = null;
    user.value = null;
    localStorage.removeItem('token');
  };

  return { user, token, isAuthenticated, isLoading, login, logout };
});
```

## API Integration

```javascript
// services/api.js
import axios from 'axios';
import { useAuthStore } from '@/stores/authStore';

const API_BASE_URL = import.meta.env.VITE_API_URL;

const apiClient = axios.create({
  baseURL: API_BASE_URL,
  headers: { 'Content-Type': 'application/json' }
});

// Add token to requests
apiClient.interceptors.request.use((config) => {
  const authStore = useAuthStore();
  if (authStore.token) {
    config.headers.Authorization = `Bearer ${authStore.token}`;
  }
  return config;
}, error => Promise.reject(error));

// Handle errors
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      const authStore = useAuthStore();
      authStore.logout();
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default apiClient;
```

## Routing

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router';
import { useAuthStore } from '@/stores/authStore';

const routes = [
  { path: '/', redirect: '/dashboard' },
  {
    path: '/login',
    component: () => import('@/views/Login.vue'),
    meta: { requiresAuth: false }
  },
  {
    path: '/dashboard',
    component: () => import('@/views/Dashboard.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/appointments',
    component: () => import('@/views/Appointments.vue'),
    meta: { requiresAuth: true }
  }
];

const router = createRouter({
  history: createWebHistory(),
  routes
});

router.beforeEach((to, from, next) => {
  const authStore = useAuthStore();
  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    next('/login');
  } else {
    next();
  }
});

export default router;
```

## Styling with Tailwind CSS

```javascript
// tailwind.config.js
export default {
  content: ["./index.html", "./src/**/*.{vue,js}"],
  theme: {
    extend: {
      colors: {
        primary: '#3B82F6',
        secondary: '#10B981'
      }
    }
  }
};
```

## Key Features

✅ **Component-based**: Modular, reusable components  
✅ **Reactive**: Vue 3's reactivity system  
✅ **Type-safe**: Proper prop validation  
✅ **Performant**: Code splitting, lazy loading  
✅ **Responsive**: Mobile-first design with Tailwind CSS  
✅ **Accessible**: WCAG compliant components  

## References

- [Vue 3 Documentation](https://vuejs.org/)
- [Vite Documentation](https://vitejs.dev/)
- [Tailwind CSS Documentation](https://tailwindcss.com/)
- [Pinia Documentation](https://pinia.vuejs.org/)
