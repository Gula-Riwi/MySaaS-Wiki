# Arquitectura Técnica - Meetlines Frontend

## Descripción General

Meetlines Frontend es una aplicación web moderna construida con **Vue 3**, **Vite**, y **Tailwind CSS**. Implementa una interfaz de usuario responsiva para que los negocios gestionen sus proyectos, servicios, citas y conversaciones en tiempo real.

## Stack Tecnológico

| Tecnología | Versión | Propósito |
|-----------|---------|----------|
| Vue.js | 3.x | Framework JavaScript reactivo |
| Vite | 5.x+ | Bundler y dev server |
| Tailwind CSS | 3.x | Framework CSS utility-first |
| JavaScript ES6+ | - | Lenguaje de programación |
| Axios | 1.x | Cliente HTTP |
| Vue Router | 4.x | Enrutamiento SPA |
| Pinia | 2.x | State management |

## Estructura del Proyecto

```
Meetlines-Frontend/
├── public/                      # Archivos estáticos
│   └── favicon.ico
├── src/
│   ├── assets/                  # Imágenes, iconos, fuentes
│   │   └── images/
│   ├── components/              # Componentes reutilizables
│   │   ├── common/              # Componentes genéricos
│   │   │   ├── Header.vue
│   │   │   ├── Sidebar.vue
│   │   │   ├── Footer.vue
│   │   │   └── Navbar.vue
│   │   ├── forms/               # Componentes de formularios
│   │   │   ├── AppointmentForm.vue
│   │   │   ├── ServiceForm.vue
│   │   │   └── LoginForm.vue
│   │   ├── cards/               # Componentes de tarjetas
│   │   │   ├── ProjectCard.vue
│   │   │   ├── ServiceCard.vue
│   │   │   └── AppointmentCard.vue
│   │   └── modals/              # Modales y diálogos
│   │       ├── ConfirmModal.vue
│   │       └── EditModal.vue
│   ├── views/                   # Vistas (páginas completas)
│   │   ├── Dashboard.vue        # Panel principal
│   │   ├── Projects.vue         # Gestión de proyectos
│   │   ├── Appointments.vue     # Gestión de citas
│   │   ├── Services.vue         # Gestión de servicios
│   │   ├── Conversations.vue    # Chat y conversaciones
│   │   ├── Feedback.vue         # Retroalimentación
│   │   ├── Settings.vue         # Configuración
│   │   └── Login.vue            # Autenticación
│   ├── router/                  # Configuración de rutas
│   │   └── index.js
│   ├── services/                # Servicios API
│   │   ├── api.js               # Configuración Axios
│   │   ├── authService.js       # Servicio de autenticación
│   │   ├── projectService.js    # Servicio de proyectos
│   │   ├── appointmentService.js # Servicio de citas
│   │   └── chatService.js       # Servicio de chat
│   ├── stores/                  # Estado global (Pinia)
│   │   ├── authStore.js         # Estado de autenticación
│   │   ├── projectStore.js      # Estado de proyectos
│   │   └── uiStore.js           # Estado de UI
│   ├── utils/                   # Utilidades y helpers
│   │   ├── validators.js        # Validadores
│   │   ├── formatters.js        # Formateadores
│   │   ├── constants.js         # Constantes
│   │   └── helpers.js           # Funciones auxiliares
│   ├── App.vue                  # Componente raíz
│   ├── main.js                  # Punto de entrada
│   └── styles.css               # Estilos globales
├── .env.example                 # Variables de entorno ejemplo
├── index.html                   # HTML principal
├── package.json                 # Dependencias del proyecto
├── vite.config.js               # Configuración de Vite
├── tailwind.config.js           # Configuración de Tailwind CSS
├── postcss.config.js            # Configuración de PostCSS
├── Dockerfile                   # Imagen Docker
├── nginx.conf                   # Configuración Nginx
└── README.md
```

## Arquitectura de Componentes

### 1. Componentes Reutilizables (Atomic Design)

#### Atoms (Componentes básicos)
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
.btn-secondary {
  @apply bg-gray-200 text-gray-800 hover:bg-gray-300;
}
</style>
```

#### Molecules (Componentes compuestos)
```vue
<!-- SearchBar.vue -->
<template>
  <div class="search-bar">
    <input
      v-model="searchQuery"
      type="text"
      placeholder="Buscar..."
      @input="handleSearch"
    />
    <Button variant="secondary" @click="clearSearch">Limpiar</Button>
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

#### Organisms (Componentes complejos)
```vue
<!-- AppointmentForm.vue -->
<template>
  <form @submit.prevent="submitForm">
    <div class="form-group">
      <label>Servicio</label>
      <ServiceSelector v-model="form.serviceId" />
    </div>

    <div class="form-group">
      <label>Cliente</label>
      <CustomerInput v-model="form.customerPhone" />
    </div>

    <div class="form-group">
      <label>Fecha y Hora</label>
      <DateTimePicker v-model="form.scheduledTime" />
    </div>

    <Button type="submit" :loading="isLoading">Crear Cita</Button>
  </form>
</template>
```

### 2. Vistas (Page Templates)

```vue
<!-- Dashboard.vue -->
<template>
  <div class="dashboard">
    <PageHeader title="Dashboard" />
    
    <div class="grid grid-cols-4 gap-4">
      <StatCard label="Citas Hoy" :value="stats.appointmentsToday" />
      <StatCard label="Conversaciones" :value="stats.conversations" />
      <StatCard label="Feedback" :value="stats.feedbackAverage" />
      <StatCard label="Ingresos" :value="stats.revenue" />
    </div>

    <div class="grid grid-cols-3 gap-4 mt-8">
      <UpcomingAppointments />
      <RecentFeedback />
      <PerformanceChart />
    </div>
  </div>
</template>

<script setup>
import { useProjectStore } from '@/stores/projectStore';
import { computed, onMounted } from 'vue';

const projectStore = useProjectStore();

const stats = computed(() => ({
  appointmentsToday: projectStore.appointmentsTodayCount,
  conversations: projectStore.conversationsCount,
  feedbackAverage: projectStore.feedbackAverage,
  revenue: projectStore.revenueToday
}));

onMounted(() => {
  projectStore.fetchDashboardStats();
});
</script>
```

## Gestión de Estado (Pinia)

### Store de Autenticación

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

  const fetchUser = async () => {
    try {
      const response = await authService.getProfile();
      user.value = response;
    } catch (error) {
      console.error('Failed to fetch user:', error);
    }
  };

  return {
    user,
    token,
    isAuthenticated,
    isLoading,
    login,
    logout,
    fetchUser
  };
});
```

### Store de Proyectos

```javascript
// stores/projectStore.js
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import * as projectService from '@/services/projectService';

export const useProjectStore = defineStore('project', () => {
  const projects = ref([]);
  const currentProject = ref(null);
  const isLoading = ref(false);

  const currentProjectData = computed(() => currentProject.value);

  const fetchProjects = async () => {
    isLoading.value = true;
    try {
      projects.value = await projectService.getProjects();
      if (projects.value.length > 0) {
        setCurrentProject(projects.value[0].id);
      }
    } finally {
      isLoading.value = false;
    }
  };

  const setCurrentProject = (projectId) => {
    currentProject.value = projects.value.find(p => p.id === projectId);
  };

  const createProject = async (projectData) => {
    try {
      const newProject = await projectService.createProject(projectData);
      projects.value.push(newProject);
      return newProject;
    } catch (error) {
      console.error('Failed to create project:', error);
      throw error;
    }
  };

  return {
    projects,
    currentProject,
    currentProjectData,
    isLoading,
    fetchProjects,
    setCurrentProject,
    createProject
  };
});
```

## Enrutamiento (Vue Router)

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router';
import { useAuthStore } from '@/stores/authStore';

const routes = [
  {
    path: '/',
    redirect: '/dashboard'
  },
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/Login.vue'),
    meta: { requiresAuth: false }
  },
  {
    path: '/register',
    name: 'Register',
    component: () => import('@/views/Register.vue'),
    meta: { requiresAuth: false }
  },
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: () => import('@/views/Dashboard.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/projects',
    name: 'Projects',
    component: () => import('@/views/Projects.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/appointments',
    name: 'Appointments',
    component: () => import('@/views/Appointments.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/services',
    name: 'Services',
    component: () => import('@/views/Services.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/conversations',
    name: 'Conversations',
    component: () => import('@/views/Conversations.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/settings',
    name: 'Settings',
    component: () => import('@/views/Settings.vue'),
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
  } else if (!to.meta.requiresAuth && authStore.isAuthenticated && to.path === '/login') {
    next('/dashboard');
  } else {
    next();
  }
});

export default router;
```

## Integración con API

```javascript
// services/api.js
import axios from 'axios';
import { useAuthStore } from '@/stores/authStore';

const API_BASE_URL = import.meta.env.VITE_API_URL || 'https://services.meet-lines.com/api';

const apiClient = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json'
  }
});

// Request interceptor para agregar token
apiClient.interceptors.request.use(
  (config) => {
    const authStore = useAuthStore();
    if (authStore.token) {
      config.headers.Authorization = `Bearer ${authStore.token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor para manejar errores
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

## Estilos y Tailwind CSS

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{vue,js,jsx,ts,tsx}"
  ],
  theme: {
    extend: {
      colors: {
        primary: '#3B82F6',
        secondary: '#10B981',
        accent: '#F59E0B'
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif']
      }
    }
  },
  plugins: []
}
```

## Patrones de Desarrollo

### 1. Composition API con Setup

```vue
<script setup>
import { ref, computed, onMounted } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();
const count = ref(0);

const doubled = computed(() => count.value * 2);

const increment = () => count.value++;

onMounted(() => {
  console.log('Componente montado');
});
</script>

<template>
  <div>
    <p>{{ count }} x 2 = {{ doubled }}</p>
    <button @click="increment">Incrementar</button>
  </div>
</template>
```

### 2. Props y Emits

```vue
<script setup>
defineProps({
  title: String,
  disabled: { type: Boolean, default: false }
});

const emit = defineEmits(['click', 'update']);

const handleClick = () => {
  emit('click');
};

const updateValue = (value) => {
  emit('update', value);
};
</script>

<template>
  <button
    :disabled="disabled"
    @click="handleClick"
  >
    {{ title }}
  </button>
</template>
```

### 3. Ciclo de vida

```javascript
import { onMounted, onUpdated, onBeforeUnmount } from 'vue';

onMounted(() => {
  // Cuando el componente se monta
  console.log('Componente montado');
});

onUpdated(() => {
  // Cuando el componente se actualiza
  console.log('Componente actualizado');
});

onBeforeUnmount(() => {
  // Antes de desmontar el componente
  console.log('Limpiando recursos');
});
```

## Rendimiento

### Code Splitting

```javascript
// Router con lazy loading
const routes = [
  {
    path: '/dashboard',
    component: () => import('@/views/Dashboard.vue')
  },
  {
    path: '/appointments',
    component: () => import('@/views/Appointments.vue')
  }
];
```

### Image Optimization

```vue
<template>
  <img
    src="@/assets/images/logo.png"
    alt="Logo"
    loading="lazy"
    width="200"
    height="200"
  />
</template>
```

## Conclusión

La arquitectura de Meetlines Frontend está diseñada para ser:

✅ **Modular**: Componentes reutilizables y independientes  
✅ **Escalable**: Fácil de agregar nuevas vistas y funcionalidades  
✅ **Mantenible**: Separación clara de responsabilidades  
✅ **Performante**: Code splitting, lazy loading, y optimizaciones  
✅ **Testeable**: Componentes puros y fáciles de testear  

## Referencias

- [Vue 3 Documentation](https://vuejs.org/)
- [Vite Documentation](https://vitejs.dev/)
- [Tailwind CSS Documentation](https://tailwindcss.com/)
- [Pinia Documentation](https://pinia.vuejs.org/)
- [Vue Router Documentation](https://router.vuejs.org/)
