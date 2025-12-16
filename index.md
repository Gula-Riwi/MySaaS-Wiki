---
title: index
description: 
published: true
date: 2025-11-27T20:40:32.609Z
tags: 
editor: markdown
dateCreated: 2025-11-26T22:37:06.556Z
---

# Wiki de MeetLines

Bienvenido a la documentación del proyecto **MeetLines**. Esta wiki está organizada en varias secciones para facilitar la comprensión de la arquitectura, el flujo de la aplicación, los bots involucrados y la documentación técnica detallada de cada repositorio.

---

## 📚 Quick Start - Documentación General

Comienza aquí para entender la arquitectura general y el flujo de la aplicación:

- [Arquitectura General](architecture.md) - Clean Architecture y patrones de diseño
- [Flujo de la Aplicación](flow.md) - Diagrama de flujos y procesos
- [Documentación VPS](MeetLines-Infrastructure/VPS-Documentation/documentacion-vps.md) - Infraestructura y deployment

---

## 🔵 Backend - ASP.NET Core

Documentación técnica completa del API principal (ASP.NET Core 8):

### Arquitectura Técnica
- **Español:** [Arquitectura Técnica - Backend](MeetLines-Backend/Technical-Architecture/arquitectura-tecnica.md)
- **English:** [Technical Architecture - Backend](MeetLines-Backend/Technical-Architecture/en/technical-architecture.md)

### Referencia de API
- **Español:** [Referencia de API](MeetLines-Backend/API-Reference/referencia-api.md)
- **English:** [API Reference](MeetLines-Backend/API-Reference/en/api-reference.md)

### Configuración y Despliegue
- **Español:** [Guía de Configuración Local](MeetLines-Backend/Setup-Guide/guia-configuracion.md)
- **English:** [Local Setup Guide](MeetLines-Backend/Setup-Guide/en/setup-guide.md)
- **Español:** [Guía de Despliegue Producción](MeetLines-Backend/Deployment-Guide/guia-despliegue.md)
- **English:** [Production Deployment Guide](MeetLines-Backend/Deployment-Guide/en/deployment-guide.md)

---

## 🟢 Frontend - Vue 3

Documentación técnica del dashboard de negocios (Vue 3 + Vite):

### Arquitectura Técnica
- **Español:** [Arquitectura Técnica - Frontend](Meetlines-Frontend/Technical-Architecture/arquitectura-tecnica.md)
- **English:** [Technical Architecture - Frontend](Meetlines-Frontend/Technical-Architecture/en/technical-architecture.md)

### Configuración y Desarrollo
- **Español:** [Guía de Configuración](Meetlines-Frontend/Setup-Guide/guia-configuracion.md)
- **English:** [Setup Guide](Meetlines-Frontend/Setup-Guide/en/setup-guide.md)

---

## 🟡 Users - Spring Boot

Documentación técnica del microservicio de autenticación y gestión de usuarios:

### Arquitectura Técnica
- **Español:** [Arquitectura Técnica - Users](MeetLines-Users/Technical-Architecture/arquitectura-tecnica.md)
- **English:** [Technical Architecture - Users](MeetLines-Users/Technical-Architecture/en/technical-architecture.md)

### Referencia de API
- **Español:** [Referencia de API - Users](MeetLines-Users/API-Reference/referencia-api.md)
- **English:** [API Reference - Users](MeetLines-Users/API-Reference/en/api-reference.md)

### Configuración
- **Español:** [Guía de Configuración](MeetLines-Users/Setup-Guide/guia-configuracion.md)
- **English:** [Setup Guide](MeetLines-Users/Setup-Guide/en/setup-guide.md)

---

## 🟣 Mobile - Android/Kotlin

Documentación técnica de la aplicación móvil (Kotlin + Jetpack Compose):

### Arquitectura Técnica
- **Español:** [Arquitectura Técnica - Mobile](MeetLines-Mobile/Technical-Architecture/arquitectura-tecnica.md)
- **English:** [Technical Architecture - Mobile](MeetLines-Mobile/Technical-Architecture/en/technical-architecture.md)

### Configuración y Desarrollo
- **Español:** [Guía de Configuración](MeetLines-Mobile/Setup-Guide/guia-configuracion.md)
- **English:** [Setup Guide](MeetLines-Mobile/Setup-Guide/en/setup-guide.md)

---

## 🧱 Infrastructure - VPS

Documentación técnica de la infraestructura del servidor y servicios compartidos:

### Documentación General
- **Español:** [Infraestructura y VPS](MeetLines-Infrastructure/VPS-Documentation/documentacion-vps.md)
- **English:** [Infrastructure and VPS](MeetLines-Infrastructure/VPS-Documentation/en/vps-documentation.md)

---

## 🗺️ Navegación Rápida

| Sección | Propósito | Acceso |
|---------|----------|--------|
| **Arquitectura** | Entender diseño general | [architecture.md](architecture.md) |
| **Flujo** | Ver procesos y workflows | [flow.md](flow.md) |
| **Backend** | ASP.NET Core API | [MeetLines-Backend/](MeetLines-Backend/Technical-Architecture/en/technical-architecture.md) |
| **Frontend** | Vue 3 Dashboard | [Meetlines-Frontend/](Meetlines-Frontend/Technical-Architecture/en/technical-architecture.md) |
| **Users** | Spring Boot Auth | [MeetLines-Users/](MeetLines-Users/Technical-Architecture/en/technical-architecture.md) |
| **Mobile** | Android App | [MeetLines-Mobile/](MeetLines-Mobile/Technical-Architecture/en/technical-architecture.md) |
| **VPS** | Infraestructura | [MeetLines-Infrastructure/](MeetLines-Infrastructure/VPS-Documentation/documentacion-vps.md) |

---

## 🚀 Para Empezar

### Según tu rol:

**👨‍💻 Backend Developer**
1. Lee [Arquitectura Técnica - Backend](MeetLines-Backend/Technical-Architecture/en/technical-architecture.md)
2. Sigue [Setup Guide - Backend](MeetLines-Backend/Setup-Guide/en/setup-guide.md)
3. Consulta [API Reference](MeetLines-Backend/API-Reference/en/api-reference.md)

**🎨 Frontend Developer**
1. Lee [Arquitectura Técnica - Frontend](Meetlines-Frontend/Technical-Architecture/en/technical-architecture.md)
2. Sigue [Setup Guide - Frontend](Meetlines-Frontend/Setup-Guide/en/setup-guide.md)

**📱 Mobile Developer**
1. Lee [Arquitectura Técnica - Mobile](MeetLines-Mobile/Technical-Architecture/en/technical-architecture.md)
2. Sigue [Setup Guide - Mobile](MeetLines-Mobile/Setup-Guide/en/setup-guide.md)

**🚀 DevOps/Infrastructure**
1. Lee [Deployment Guide](MeetLines-Backend/Deployment-Guide/en/deployment-guide.md)
2. Consulta [VPS Documentation](MeetLines-Infrastructure/VPS-Documentation/en/vps-documentation.md)

---

## 📖 Estructura de Documentación

Cada repositorio contiene:

```
Repository/
├── Technical-Architecture/
│   ├── arquitectura-tecnica.md      (Español)
│   └── en/technical-architecture.md (English)
├── API-Reference/ (si aplica)
│   ├── referencia-api.md            (Español)
│   └── en/api-reference.md          (English)
├── Setup-Guide/
│   ├── guia-configuracion.md        (Español)
│   └── en/setup-guide.md            (English)
└── Deployment-Guide/ (si aplica)
    ├── guia-despliegue.md           (Español)
    └── en/deployment-guide.md       (English)
```

---

Utiliza los enlaces anteriores para navegar a la sección correspondiente.
