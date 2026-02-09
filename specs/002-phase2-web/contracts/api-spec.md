# API Contract Specification

**Feature**: 002-phase2-web
**Base URL**: `http://localhost:8000/api`
**Protocol**: HTTP/1.1
**Content-Type**: `application/json`
**Authentication**: JWT in httpOnly cookie

---

## Authentication Endpoints

### POST /api/auth/register

Register a new user account.

**Request**:
```http
POST /api/auth/register HTTP/1.1
Host: localhost:8000
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Request Schema**:
```json
{
  "email": "string (EmailStr, required)",
  "password": "string (min 8 chars, required)"
}
```

**Success Response** (201 Created):
```http
HTTP/1.1 201 Created
Content-Type: application/json
Set-Cookie: access_token=<jwt>; HttpOnly; SameSite=Lax; Max-Age=86400; Path=/

{
  "id": 1,
  "email": "user@example.com",
  "created_at": "2026-01-01T10:30:00Z"
}
```

**Error Responses**:

*400 Bad Request - Invalid Email*:
```json
{
  "detail": "Invalid email format"
}
```

*400 Bad Request - Password Too Short*:
```json
{
  "detail": "Password must be at least 8 characters"
}
```

*409 Conflict - Email Already Exists*:
```json
{
  "detail": "Email already registered"
}
```

---

### POST /api/auth/login

Authenticate existing user and issue JWT token.

**Request**:
```http
POST /api/auth/login HTTP/1.1
Host: localhost:8000
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Request Schema**:
```json
{
  "email": "string (EmailStr, required)",
  "password": "string (required)"
}
```

**Success Response** (200 OK):
```http
HTTP/1.1 200 OK
Content-Type: application/json
Set-Cookie: access_token=<jwt>; HttpOnly; SameSite=Lax; Max-Age=86400; Path=/

{
  "id": 1,
  "email": "user@example.com",
  "created_at": "2026-01-01T10:30:00Z"
}
```

**Error Responses**:

*401 Unauthorized - Invalid Credentials*:
```json
{
  "detail": "Invalid credentials"
}
```

**Note**: Same error message for wrong email or wrong password (prevents user enumeration).

---

### POST /api/auth/logout

Clear authentication token and logout user.

**Request**:
```http
POST /api/auth/logout HTTP/1.1
Host: localhost:8000
Cookie: access_token=<jwt>
```

**Success Response** (200 OK):
```http
HTTP/1.1 200 OK
Content-Type: application/json
Set-Cookie: access_token=; HttpOnly; SameSite=Lax; Max-Age=0; Path=/

{
  "message": "Successfully logged out"
}
```

**Note**: Works even without valid token (idempotent operation).

---

## Task Endpoints

**All task endpoints require authentication via JWT cookie.**

### GET /api/tasks

Get all tasks for authenticated user.

**Request**:
```http
GET /api/tasks HTTP/1.1
Host: localhost:8000
Cookie: access_token=<jwt>
```

**Success Response** (200 OK):
```http
HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 1,
    "user_id": 1,
    "title": "Buy groceries",
    "description": "Milk, eggs, bread",
    "priority": "high",
    "is_complete": false,
    "created_at": "2026-01-01T10:35:00Z",
    "updated_at": "2026-01-01T10:35:00Z"
  },
  {
    "id": 2,
    "user_id": 1,
    "title": "Call dentist",
    "description": "",
    "priority": "medium",
    "is_complete": true,
    "created_at": "2026-01-01T11:00:00Z",
    "updated_at": "2026-01-01T14:30:00Z"
  }
]
```

**Empty List Response**:
```json
[]
```

**Error Responses**:

*401 Unauthorized - No Token*:
```json
{
  "detail": "Not authenticated"
}
```

*401 Unauthorized - Invalid Token*:
```json
{
  "detail": "Invalid token"
}
```

---

### POST /api/tasks

Create a new task for authenticated user.

**Request**:
```http
POST /api/tasks HTTP/1.1
Host: localhost:8000
Cookie: access_token=<jwt>
Content-Type: application/json

{
  "title": "Buy groceries",
  "description": "Milk, eggs, bread, coffee",
  "priority": "high"
}
```

**Request Schema**:
```json
{
  "title": "string (required, min 1 char, max 200 chars)",
  "description": "string (optional, default '', max 2000 chars)",
  "priority": "string (optional, default 'medium', enum: high/medium/low)"
}
```

**Success Response** (201 Created):
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 1,
  "user_id": 1,
  "title": "Buy groceries",
  "description": "Milk, eggs, bread, coffee",
  "priority": "high",
  "is_complete": false,
  "created_at": "2026-01-01T10:35:00Z",
  "updated_at": "2026-01-01T10:35:00Z"
}
```

**Error Responses**:

*400 Bad Request - Empty Title*:
```json
{
  "detail": "Title is required"
}
```

*400 Bad Request - Invalid Priority*:
```json
{
  "detail": "Priority must be one of: high, medium, low"
}
```

*401 Unauthorized*:
```json
{
  "detail": "Not authenticated"
}
```

---

### GET /api/tasks/{task_id}

Get a specific task by ID (must belong to authenticated user).

**Request**:
```http
GET /api/tasks/1 HTTP/1.1
Host: localhost:8000
Cookie: access_token=<jwt>
```

**Success Response** (200 OK):
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 1,
  "user_id": 1,
  "title": "Buy groceries",
  "description": "Milk, eggs, bread, coffee",
  "priority": "high",
  "is_complete": false,
  "created_at": "2026-01-01T10:35:00Z",
  "updated_at": "2026-01-01T10:35:00Z"
}
```

**Error Responses**:

*404 Not Found*:
```json
{
  "detail": "Task not found"
}
```

*403 Forbidden - Task Belongs to Another User*:
```json
{
  "detail": "Not authorized to access this task"
}
```

*401 Unauthorized*:
```json
{
  "detail": "Not authenticated"
}
```

---

### PUT /api/tasks/{task_id}

Update a task's title, description, or priority.

**Request**:
```http
PUT /api/tasks/1 HTTP/1.1
Host: localhost:8000
Cookie: access_token=<jwt>
Content-Type: application/json

{
  "title": "Buy almond milk",
  "description": "Unsweetened almond milk",
  "priority": "medium"
}
```

**Request Schema**:
```json
{
  "title": "string (optional, min 1 char if provided, max 200 chars)",
  "description": "string (optional, max 2000 chars)",
  "priority": "string (optional, enum: high/medium/low)"
}
```

**Note**: All fields are optional. Only provided fields are updated.

**Success Response** (200 OK):
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 1,
  "user_id": 1,
  "title": "Buy almond milk",
  "description": "Unsweetened almond milk",
  "priority": "medium",
  "is_complete": false,
  "created_at": "2026-01-01T10:35:00Z",
  "updated_at": "2026-01-01T15:20:00Z"
}
```

**Error Responses**:

*400 Bad Request - Empty Title*:
```json
{
  "detail": "Title cannot be empty"
}
```

*404 Not Found*:
```json
{
  "detail": "Task not found"
}
```

*403 Forbidden*:
```json
{
  "detail": "Not authorized to update this task"
}
```

*401 Unauthorized*:
```json
{
  "detail": "Not authenticated"
}
```

---

### PATCH /api/tasks/{task_id}/toggle

Toggle task completion status (incomplete ↔ complete).

**Request**:
```http
PATCH /api/tasks/1/toggle HTTP/1.1
Host: localhost:8000
Cookie: access_token=<jwt>
```

**No request body required.**

**Success Response** (200 OK):
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 1,
  "user_id": 1,
  "title": "Buy groceries",
  "description": "Milk, eggs, bread, coffee",
  "priority": "high",
  "is_complete": true,
  "created_at": "2026-01-01T10:35:00Z",
  "updated_at": "2026-01-01T16:00:00Z"
}
```

**Note**: `is_complete` is toggled from `false` → `true` or `true` → `false`.

**Error Responses**:

*404 Not Found*:
```json
{
  "detail": "Task not found"
}
```

*403 Forbidden*:
```json
{
  "detail": "Not authorized to update this task"
}
```

*401 Unauthorized*:
```json
{
  "detail": "Not authenticated"
}
```

---

### DELETE /api/tasks/{task_id}

Permanently delete a task.

**Request**:
```http
DELETE /api/tasks/1 HTTP/1.1
Host: localhost:8000
Cookie: access_token=<jwt>
```

**Success Response** (204 No Content):
```http
HTTP/1.1 204 No Content
```

**Alternative Success Response** (200 OK with confirmation):
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "message": "Task deleted successfully"
}
```

**Error Responses**:

*404 Not Found*:
```json
{
  "detail": "Task not found"
}
```

*403 Forbidden*:
```json
{
  "detail": "Not authorized to delete this task"
}
```

*401 Unauthorized*:
```json
{
  "detail": "Not authenticated"
}
```

---

## HTTP Status Codes

### Success Codes
- **200 OK**: Request succeeded (GET, PUT, PATCH, DELETE with body)
- **201 Created**: Resource created successfully (POST /auth/register, POST /tasks)
- **204 No Content**: Request succeeded with no response body (DELETE)

### Client Error Codes
- **400 Bad Request**: Invalid input (validation failed)
- **401 Unauthorized**: Missing or invalid authentication token
- **403 Forbidden**: Valid token but insufficient permissions (e.g., accessing another user's task)
- **404 Not Found**: Resource does not exist
- **409 Conflict**: Resource conflict (e.g., duplicate email)
- **422 Unprocessable Entity**: Validation error (alternative to 400)

### Server Error Codes
- **500 Internal Server Error**: Unexpected server error
- **503 Service Unavailable**: Database connection failed

---

## Authentication Flow

### Registration Flow
```
Client                          Backend                      Database
  |                               |                              |
  |-- POST /auth/register ------->|                              |
  |   {email, password}           |                              |
  |                               |-- Check email unique ------->|
  |                               |<-- Email available ----------|
  |                               |-- Hash password (bcrypt) ----|
  |                               |-- INSERT user --------------->|
  |                               |<-- User created --------------|
  |                               |-- Generate JWT ---------------|
  |<-- 201 Created + Set-Cookie --|                              |
  |    {id, email, created_at}    |                              |
```

### Login Flow
```
Client                          Backend                      Database
  |                               |                              |
  |-- POST /auth/login ---------->|                              |
  |   {email, password}           |                              |
  |                               |-- SELECT user by email ----->|
  |                               |<-- User record --------------|
  |                               |-- Verify password (bcrypt) --|
  |                               |-- Generate JWT ---------------|
  |<-- 200 OK + Set-Cookie -------|                              |
  |    {id, email, created_at}    |                              |
```

### Protected Request Flow
```
Client                          Backend                      Database
  |                               |                              |
  |-- GET /tasks ---------------->|                              |
  |   Cookie: access_token=<jwt>  |                              |
  |                               |-- Decode JWT ----------------|
  |                               |-- Validate signature ---------|
  |                               |-- Extract user_id ------------|
  |                               |-- SELECT tasks WHERE -------->|
  |                               |   user_id = <from_jwt>        |
  |                               |<-- Tasks list ---------------|
  |<-- 200 OK --------------------|                              |
  |    [{task1}, {task2}, ...]    |                              |
```

---

## CORS Configuration

**Allowed Origins**:
- `http://localhost:3000` (Next.js dev server)
- Production frontend URL (to be configured)

**Allowed Methods**:
- GET, POST, PUT, PATCH, DELETE, OPTIONS

**Allowed Headers**:
- Content-Type, Authorization

**Allow Credentials**:
- `true` (required for cookies)

**Preflight Caching**:
- `max-age: 86400` (24 hours)

---

## Rate Limiting (Future Enhancement)

**Not implemented in MVP, but recommended for production**:

- Login attempts: 5 per minute per IP
- Registration: 3 per hour per IP
- API requests: 100 per minute per user
- Global: 1000 requests per minute

---

## Error Response Format

**All errors follow consistent format**:

```json
{
  "detail": "Human-readable error message"
}
```

**Validation errors** (422 Unprocessable Entity):
```json
{
  "detail": [
    {
      "loc": ["body", "email"],
      "msg": "value is not a valid email address",
      "type": "value_error.email"
    }
  ]
}
```

---

## Security Headers

**All responses include**:
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `X-XSS-Protection: 1; mode=block`
- `Strict-Transport-Security: max-age=31536000` (HTTPS only, production)

---

## OpenAPI Documentation

**Auto-generated documentation available at**:
- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`
- OpenAPI JSON: `http://localhost:8000/openapi.json`

---

## Example cURL Commands

### Register
```bash
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"SecurePass123!"}' \
  -c cookies.txt
```

### Login
```bash
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"SecurePass123!"}' \
  -c cookies.txt
```

### Get Tasks
```bash
curl -X GET http://localhost:8000/api/tasks \
  -b cookies.txt
```

### Create Task
```bash
curl -X POST http://localhost:8000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Buy groceries","description":"Milk, eggs","priority":"high"}' \
  -b cookies.txt
```

### Update Task
```bash
curl -X PUT http://localhost:8000/api/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"title":"Buy almond milk"}' \
  -b cookies.txt
```

### Toggle Task
```bash
curl -X PATCH http://localhost:8000/api/tasks/1/toggle \
  -b cookies.txt
```

### Delete Task
```bash
curl -X DELETE http://localhost:8000/api/tasks/1 \
  -b cookies.txt
```

### Logout
```bash
curl -X POST http://localhost:8000/api/auth/logout \
  -b cookies.txt
```
