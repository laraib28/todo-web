---
name: api-testing
description: Test FastAPI backend endpoints
---

# API Testing

## Run backend
```bash
cd backend && uvicorn app.main:app --reload
```

## Test endpoints
```bash
# Register
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"test","email":"test@test.com","password":"pass123"}'

# Login
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"pass123"}'

# List tasks (with token)
curl http://localhost:8000/api/tasks \
  -H "Authorization: Bearer <token>"
```
