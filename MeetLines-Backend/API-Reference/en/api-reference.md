# API Reference - MeetLines Backend

## Introduction

MeetLines API provides RESTful endpoints for managing projects, appointments, services, conversations and more. All endpoints require authentication via JWT Bearer Token.

**Base URL:** `https://services.meet-lines.com/api`  
**Interactive Documentation:** `https://services.meet-lines.com/swagger`

## Authentication

All protected endpoints require an `Authorization` header:

```bash
curl -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  https://services.meet-lines.com/api/projects
```

### Getting a Token

#### OAuth Discord
```http
POST /api/Auth/oauth/discord
Content-Type: application/json

{
  "code": "discord_authorization_code",
  "redirectUri": "https://yourdomain.com/callback"
}
```

**Response (200):**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "refresh_token_here",
  "expiresIn": 3600
}
```

#### Email Login
```http
POST /api/Auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "secure_password"
}
```

#### Register
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

## Projects

### List Projects

```http
GET /api/Projects
Authorization: Bearer TOKEN
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-----------|
| page | integer | Page number (default: 1) |
| pageSize | integer | Items per page (default: 10) |

**Response (200):**
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "My Business",
      "description": "Business description",
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

### Get a Project

```http
GET /api/Projects/{projectId}
Authorization: Bearer TOKEN
```

**Response (200):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "My Business",
  "description": "Business description",
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

### Create Project

```http
POST /api/Projects
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "name": "My Business",
  "description": "Business description",
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

**Response (201):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "My Business",
  "status": "active",
  "createdAt": "2025-01-15T10:30:00Z"
}
```

### Update Project

```http
PUT /api/Projects/{projectId}
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "name": "My Updated Business",
  "description": "New description",
  "email": "newemail@business.com",
  "settings": {
    "allowOnlineBooking": true,
    "autoConfirmAppointments": true
  }
}
```

---

## Services

### List Services

```http
GET /api/projects/{projectId}/services
Authorization: Bearer TOKEN
```

**Response (200):**
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440001",
      "projectId": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Haircut",
      "description": "Classic men's haircut",
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

### Create Service

```http
POST /api/projects/{projectId}/services
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "name": "Haircut",
  "description": "Classic men's haircut",
  "price": 25.00,
  "duration": 30,
  "durationUnit": "minutes",
  "maxCapacity": 1
}
```

### Update Service

```http
PUT /api/services/{serviceId}
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "name": "Premium Haircut",
  "price": 35.00,
  "duration": 45
}
```

### Delete Service

```http
DELETE /api/services/{serviceId}
Authorization: Bearer TOKEN
```

---

## Appointments

### List Appointments

```http
GET /api/projects/{projectId}/appointments
Authorization: Bearer TOKEN
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-----------|
| status | string | Filter by status (pending, confirmed, completed, cancelled) |
| serviceId | uuid | Filter by service |
| startDate | date | From date (YYYY-MM-DD) |
| endDate | date | To date (YYYY-MM-DD) |

**Response (200):**
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440002",
      "projectId": "550e8400-e29b-41d4-a716-446655440000",
      "serviceId": "550e8400-e29b-41d4-a716-446655440001",
      "customerName": "John Smith",
      "customerPhone": "+34612345678",
      "scheduledTime": "2025-02-15T14:00:00Z",
      "duration": 30,
      "status": "confirmed",
      "notes": "New customer",
      "createdAt": "2025-01-15T10:30:00Z"
    }
  ],
  "totalCount": 1
}
```

### Get Available Slots

```http
GET /api/projects/{projectId}/appointments/available-slots
Authorization: Bearer TOKEN
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-----------|
| serviceId | uuid | Service ID (required) |
| date | date | Desired date (YYYY-MM-DD, required) |

**Response (200):**
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

### Create Appointment

```http
POST /api/projects/{projectId}/appointments
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "serviceId": "550e8400-e29b-41d4-a716-446655440001",
  "customerName": "John Smith",
  "customerPhone": "+34612345678",
  "customerEmail": "john@example.com",
  "scheduledTime": "2025-02-15T14:00:00Z",
  "notes": "New customer"
}
```

**Response (201):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440002",
  "status": "pending",
  "confirmationCode": "APT-2025-001",
  "createdAt": "2025-01-15T10:30:00Z"
}
```

### Update Appointment Status

```http
PATCH /api/projects/{projectId}/appointments/{id}/status
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "status": "confirmed",
  "notes": "Confirmed by customer"
}
```

**Valid Statuses:**
- `pending` - Pending confirmation
- `confirmed` - Confirmed
- `completed` - Completed
- `cancelled` - Cancelled

---

## Conversations

### List Conversations

```http
GET /api/projects/{projectId}/conversations
Authorization: Bearer TOKEN
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-----------|
| status | string | pending, handled, closed |
| sortBy | string | createdAt, updatedAt |

**Response (200):**
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440003",
      "projectId": "550e8400-e29b-41d4-a716-446655440000",
      "customerPhone": "+34612345678",
      "customerName": "Mary López",
      "channel": "whatsapp",
      "status": "pending",
      "lastMessage": "What are your hours?",
      "unreadMessages": 1,
      "createdAt": "2025-01-20T10:00:00Z",
      "updatedAt": "2025-01-20T14:30:00Z"
    }
  ]
}
```

### Mark Conversation as Handled

```http
POST /api/projects/{projectId}/conversations/{id}/mark-handled
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "notes": "Conversation resolved successfully"
}
```

### Get Chat History

```http
GET /api/projects/{projectId}/chat/{phone}/history
Authorization: Bearer TOKEN
```

**Response (200):**
```json
{
  "messages": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440004",
      "content": "What are your hours?",
      "sender": "customer",
      "timestamp": "2025-01-20T10:00:00Z",
      "messageType": "text"
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440005",
      "content": "We're open 9 AM to 8 PM",
      "sender": "business",
      "timestamp": "2025-01-20T10:05:00Z",
      "messageType": "text"
    }
  ],
  "totalMessages": 2
}
```

---

## Feedback

### List Feedback

```http
GET /api/projects/{projectId}/feedback
Authorization: Bearer TOKEN
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-----------|
| ratingFrom | integer | Minimum rating (1-5) |
| ratingTo | integer | Maximum rating (1-5) |
| responded | boolean | Show only responded/unanswered |

**Response (200):**
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440006",
      "projectId": "550e8400-e29b-41d4-a716-446655440000",
      "customerName": "John Smith",
      "rating": 5,
      "comment": "Excellent service",
      "appointmentId": "550e8400-e29b-41d4-a716-446655440002",
      "responded": false,
      "createdAt": "2025-01-20T16:00:00Z"
    }
  ],
  "averageRating": 4.8,
  "totalFeedback": 1
}
```

### Create Feedback (Client)

```http
POST /api/projects/{projectId}/feedback
Content-Type: application/json

{
  "appointmentId": "550e8400-e29b-41d4-a716-446655440002",
  "customerName": "John Smith",
  "customerPhone": "+34612345678",
  "rating": 5,
  "comment": "Excellent service"
}
```

### Respond to Feedback

```http
POST /api/projects/{projectId}/feedback/{id}/respond
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "response": "Thank you for your rating! We look forward to seeing you soon."
}
```

---

## HTTP Status Codes

| Code | Description |
|------|-----------|
| 200 | OK - Successful request |
| 201 | Created - Resource created successfully |
| 204 | No Content - Successful request with no content |
| 400 | Bad Request - Invalid input data |
| 401 | Unauthorized - Missing or invalid token |
| 403 | Forbidden - No permission to access resource |
| 404 | Not Found - Resource not found |
| 409 | Conflict - Conflict (e.g., duplicate appointment) |
| 422 | Unprocessable Entity - Invalid entity |
| 500 | Internal Server Error - Server error |

## Error Response Format

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

The API implements rate limiting to prevent abuse:

- **Limit:** 100 requests per minute per user
- **Response Header:** `X-RateLimit-Remaining: 99`

If you exceed the limit, you'll receive a `429 Too Many Requests` error.

## Pagination

For endpoints returning lists, use these parameters:

```http
GET /api/projects?page=1&pageSize=20
```

**Response includes:**
```json
{
  "data": [...],
  "totalCount": 100,
  "pageNumber": 1,
  "pageSize": 20,
  "totalPages": 5
}
```

## Webhooks (Coming Soon)

Receive real-time notifications about events:

- `appointment.created`
- `appointment.confirmed`
- `appointment.completed`
- `feedback.received`
- `conversation.new`

Contact the team to enable webhooks on your account.

## cURL Examples

### Get All Projects
```bash
curl -X GET "https://services.meet-lines.com/api/Projects" \
  -H "Authorization: Bearer your_token"
```

### Create an Appointment
```bash
curl -X POST "https://services.meet-lines.com/api/projects/{projectId}/appointments" \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "serviceId": "550e8400-e29b-41d4-a716-446655440001",
    "customerName": "John Smith",
    "customerPhone": "+34612345678",
    "scheduledTime": "2025-02-15T14:00:00Z"
  }'
```

### List Appointments with Filters
```bash
curl -X GET "https://services.meet-lines.com/api/projects/{projectId}/appointments?status=confirmed&startDate=2025-02-01&endDate=2025-02-28" \
  -H "Authorization: Bearer your_token"
```

## Support and Documentation

- **Swagger UI:** https://services.meet-lines.com/swagger
- **Email:** support@meet-lines.com
- **Full Documentation:** https://docs.meet-lines.com
