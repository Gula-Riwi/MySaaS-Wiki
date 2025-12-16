# Referencia de API - MeetLines Backend

## Introducción

MeetLines API proporciona endpoints RESTful para gestionar proyectos, citas, servicios, conversaciones y más. Todos los endpoints requieren autenticación mediante JWT Bearer Token.

**URL Base:** `https://services.meet-lines.com/api`  
**Documentación interactiva:** `https://services.meet-lines.com/swagger`

## Autenticación

Todos los endpoints protegidos requieren un header `Authorization`:

```bash
curl -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  https://services.meet-lines.com/api/projects
```

### Obtener Token

#### OAuth Discord
```http
POST /api/Auth/oauth/discord
Content-Type: application/json

{
  "code": "discord_authorization_code",
  "redirectUri": "https://yourdomain.com/callback"
}
```

**Respuesta (200):**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "refresh_token_here",
  "expiresIn": 3600
}
```

#### Login con Email
```http
POST /api/Auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "secure_password"
}
```

#### Registro
```http
POST /api/Auth/register
Content-Type: application/json

{
  "email": "newuser@example.com",
  "password": "secure_password",
  "firstName": "John",
  "lastName": "Doe",
  "businessName": "My Business"
}
```

#### Refresh Token
```http
POST /api/Auth/refresh-token
Content-Type: application/json

{
  "refreshToken": "your_refresh_token"
}
```

---

## Proyectos (Projects)

### Listar proyectos

```http
GET /api/Projects
Authorization: Bearer TOKEN
```

**Parámetros de query:**
| Parámetro | Tipo | Descripción |
|-----------|------|-----------|
| page | integer | Número de página (default: 1) |
| pageSize | integer | Items por página (default: 10) |

**Respuesta (200):**
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Mi Negocio",
      "description": "Descripción del negocio",
      "businessType": "barbershop",
      "phoneNumber": "+34123456789",
      "email": "contact@business.com",
      "website": "https://business.com",
      "status": "active",
      "createdAt": "2025-01-15T10:30:00Z"
    }
  ],
  "totalCount": 1,
  "pageNumber": 1,
  "pageSize": 10
}
```

### Obtener un proyecto

```http
GET /api/Projects/{projectId}
Authorization: Bearer TOKEN
```

**Respuesta (200):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Mi Negocio",
  "description": "Descripción del negocio",
  "businessType": "barbershop",
  "phoneNumber": "+34123456789",
  "email": "contact@business.com",
  "timezone": "Europe/Madrid",
  "status": "active",
  "settings": {
    "allowOnlineBooking": true,
    "autoConfirmAppointments": false,
    "sendReminders": true
  },
  "createdAt": "2025-01-15T10:30:00Z",
  "updatedAt": "2025-01-20T14:20:00Z"
}
```

### Crear proyecto

```http
POST /api/Projects
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "name": "Mi Negocio",
  "description": "Descripción del negocio",
  "businessType": "barbershop",
  "phoneNumber": "+34123456789",
  "email": "contact@business.com",
  "website": "https://business.com",
  "timezone": "Europe/Madrid",
  "settings": {
    "allowOnlineBooking": true,
    "autoConfirmAppointments": false,
    "sendReminders": true
  }
}
```

**Respuesta (201):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Mi Negocio",
  "status": "active",
  "createdAt": "2025-01-15T10:30:00Z"
}
```

### Actualizar proyecto

```http
PUT /api/Projects/{projectId}
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "name": "Mi Negocio Actualizado",
  "description": "Nueva descripción",
  "email": "newemail@business.com",
  "settings": {
    "allowOnlineBooking": true,
    "autoConfirmAppointments": true
  }
}
```

---

## Servicios (Services)

### Listar servicios de un proyecto

```http
GET /api/projects/{projectId}/services
Authorization: Bearer TOKEN
```

**Respuesta (200):**
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440001",
      "projectId": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Corte de Cabello",
      "description": "Corte clásico para hombre",
      "price": 25.00,
      "duration": 30,
      "durationUnit": "minutes",
      "maxCapacity": 1,
      "status": "active",
      "createdAt": "2025-01-15T10:30:00Z"
    }
  ]
}
```

### Crear servicio

```http
POST /api/projects/{projectId}/services
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "name": "Corte de Cabello",
  "description": "Corte clásico para hombre",
  "price": 25.00,
  "duration": 30,
  "durationUnit": "minutes",
  "maxCapacity": 1
}
```

### Actualizar servicio

```http
PUT /api/services/{serviceId}
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "name": "Corte Premium",
  "price": 35.00,
  "duration": 45
}
```

### Eliminar servicio

```http
DELETE /api/services/{serviceId}
Authorization: Bearer TOKEN
```

---

## Citas (Appointments)

### Listar citas de un proyecto

```http
GET /api/projects/{projectId}/appointments
Authorization: Bearer TOKEN
```

**Parámetros de query:**
| Parámetro | Tipo | Descripción |
|-----------|------|-----------|
| status | string | Filtrar por estado (pending, confirmed, completed, cancelled) |
| serviceId | uuid | Filtrar por servicio |
| startDate | date | Desde (YYYY-MM-DD) |
| endDate | date | Hasta (YYYY-MM-DD) |

**Respuesta (200):**
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440002",
      "projectId": "550e8400-e29b-41d4-a716-446655440000",
      "serviceId": "550e8400-e29b-41d4-a716-446655440001",
      "customerName": "Juan García",
      "customerPhone": "+34612345678",
      "scheduledTime": "2025-02-15T14:00:00Z",
      "duration": 30,
      "status": "confirmed",
      "notes": "Cliente nuevo",
      "createdAt": "2025-01-15T10:30:00Z"
    }
  ],
  "totalCount": 1
}
```

### Obtener ranuras de tiempo disponibles

```http
GET /api/projects/{projectId}/appointments/available-slots
Authorization: Bearer TOKEN
```

**Parámetros de query:**
| Parámetro | Tipo | Descripción |
|-----------|------|-----------|
| serviceId | uuid | ID del servicio (requerido) |
| date | date | Fecha deseada (YYYY-MM-DD, requerido) |

**Respuesta (200):**
```json
{
  "availableSlots": [
    {
      "startTime": "2025-02-15T09:00:00Z",
      "endTime": "2025-02-15T09:30:00Z"
    },
    {
      "startTime": "2025-02-15T09:30:00Z",
      "endTime": "2025-02-15T10:00:00Z"
    }
  ]
}
```

### Crear cita

```http
POST /api/projects/{projectId}/appointments
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "serviceId": "550e8400-e29b-41d4-a716-446655440001",
  "customerName": "Juan García",
  "customerPhone": "+34612345678",
  "customerEmail": "juan@example.com",
  "scheduledTime": "2025-02-15T14:00:00Z",
  "notes": "Cliente nuevo"
}
```

**Respuesta (201):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440002",
  "status": "pending",
  "confirmationCode": "APT-2025-001",
  "createdAt": "2025-01-15T10:30:00Z"
}
```

### Actualizar estado de cita

```http
PATCH /api/projects/{projectId}/appointments/{id}/status
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "status": "confirmed",
  "notes": "Confirmado por cliente"
}
```

**Estados válidos:**
- `pending` - Pendiente de confirmación
- `confirmed` - Confirmada
- `completed` - Completada
- `cancelled` - Cancelada

---

## Conversaciones (Conversations)

### Listar conversaciones

```http
GET /api/projects/{projectId}/conversations
Authorization: Bearer TOKEN
```

**Parámetros de query:**
| Parámetro | Tipo | Descripción |
|-----------|------|-----------|
| status | string | pending, handled, closed |
| sortBy | string | createdAt, updatedAt |

**Respuesta (200):**
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440003",
      "projectId": "550e8400-e29b-41d4-a716-446655440000",
      "customerPhone": "+34612345678",
      "customerName": "María López",
      "channel": "whatsapp",
      "status": "pending",
      "lastMessage": "¿Cuál es tu horario?",
      "unreadMessages": 1,
      "createdAt": "2025-01-20T10:00:00Z",
      "updatedAt": "2025-01-20T14:30:00Z"
    }
  ]
}
```

### Marcar conversación como manejada

```http
POST /api/projects/{projectId}/conversations/{id}/mark-handled
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "notes": "Conversación resuelta con éxito"
}
```

### Obtener historial de chat

```http
GET /api/projects/{projectId}/chat/{phone}/history
Authorization: Bearer TOKEN
```

**Respuesta (200):**
```json
{
  "messages": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440004",
      "content": "¿Cuál es tu horario?",
      "sender": "customer",
      "timestamp": "2025-01-20T10:00:00Z",
      "messageType": "text"
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440005",
      "content": "Abierto de 9 AM a 8 PM",
      "sender": "business",
      "timestamp": "2025-01-20T10:05:00Z",
      "messageType": "text"
    }
  ],
  "totalMessages": 2
}
```

---

## Retroalimentación (Feedback)

### Listar feedback de un proyecto

```http
GET /api/projects/{projectId}/feedback
Authorization: Bearer TOKEN
```

**Parámetros de query:**
| Parámetro | Tipo | Descripción |
|-----------|------|-----------|
| ratingFrom | integer | Calificación mínima (1-5) |
| ratingTo | integer | Calificación máxima (1-5) |
| responded | boolean | Mostrar solo respondidos/sin responder |

**Respuesta (200):**
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440006",
      "projectId": "550e8400-e29b-41d4-a716-446655440000",
      "customerName": "Juan García",
      "rating": 5,
      "comment": "Excelente servicio",
      "appointmentId": "550e8400-e29b-41d4-a716-446655440002",
      "responded": false,
      "createdAt": "2025-01-20T16:00:00Z"
    }
  ],
  "averageRating": 4.8,
  "totalFeedback": 1
}
```

### Crear feedback (cliente)

```http
POST /api/projects/{projectId}/feedback
Content-Type: application/json

{
  "appointmentId": "550e8400-e29b-41d4-a716-446655440002",
  "customerName": "Juan García",
  "customerPhone": "+34612345678",
  "rating": 5,
  "comment": "Excelente servicio"
}
```

### Responder a feedback

```http
POST /api/projects/{projectId}/feedback/{id}/respond
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "response": "¡Gracias por tu valoración! Esperamos verte pronto."
}
```

---

## Codigos de Estado HTTP

| Código | Descripción |
|--------|-----------|
| 200 | OK - Solicitud exitosa |
| 201 | Created - Recurso creado exitosamente |
| 204 | No Content - Solicitud exitosa sin contenido |
| 400 | Bad Request - Datos de entrada inválidos |
| 401 | Unauthorized - Token faltante o inválido |
| 403 | Forbidden - Sin permiso para acceder al recurso |
| 404 | Not Found - Recurso no encontrado |
| 409 | Conflict - Conflicto (ej: cita duplicada) |
| 422 | Unprocessable Entity - Entidad no válida |
| 500 | Internal Server Error - Error del servidor |

## Formatos de Respuesta de Error

```json
{
  "status": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Email must be valid"
    },
    {
      "field": "password",
      "message": "Password must be at least 8 characters"
    }
  ],
  "timestamp": "2025-01-20T14:30:00Z"
}
```

## Rate Limiting

La API implementa rate limiting para prevenir abuso:

- **Límite:** 100 requests por minuto por usuario
- **Header de respuesta:** `X-RateLimit-Remaining: 99`

Si excedes el límite, recibirás un error `429 Too Many Requests`.

## Paginación

Para endpoints que devuelven listas, usa estos parámetros:

```http
GET /api/projects?page=1&pageSize=20
```

**Respuesta incluye:**
```json
{
  "data": [...],
  "totalCount": 100,
  "pageNumber": 1,
  "pageSize": 20,
  "totalPages": 5
}
```

## Webhooks (Próximamente)

Recibe notificaciones en tiempo real sobre eventos:

- `appointment.created`
- `appointment.confirmed`
- `appointment.completed`
- `feedback.received`
- `conversation.new`

Contacta al equipo para activar webhooks en tu cuenta.

## Ejemplos con cURL

### Obtener todos los proyectos
```bash
curl -X GET "https://services.meet-lines.com/api/Projects" \
  -H "Authorization: Bearer your_token"
```

### Crear una cita
```bash
curl -X POST "https://services.meet-lines.com/api/projects/{projectId}/appointments" \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "serviceId": "550e8400-e29b-41d4-a716-446655440001",
    "customerName": "Juan García",
    "customerPhone": "+34612345678",
    "scheduledTime": "2025-02-15T14:00:00Z"
  }'
```

### Listar citas con filtros
```bash
curl -X GET "https://services.meet-lines.com/api/projects/{projectId}/appointments?status=confirmed&startDate=2025-02-01&endDate=2025-02-28" \
  -H "Authorization: Bearer your_token"
```

## Soporte y Documentación

- **Swagger UI:** https://services.meet-lines.com/swagger
- **Email:** support@meet-lines.com
- **Documentación completa:** https://docs.meet-lines.com
