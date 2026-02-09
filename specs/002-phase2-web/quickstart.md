# Quickstart Guide: Phase-II Full-Stack Web Todo Application

**Feature**: 002-phase2-web
**Last Updated**: 2026-01-01
**Prerequisites**: Python 3.11+, Node.js 20+, Neon PostgreSQL account

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Backend Setup](#backend-setup)
3. [Frontend Setup](#frontend-setup)
4. [Database Setup](#database-setup)
5. [Running the Application](#running-the-application)
6. [Testing the API](#testing-the-api)
7. [Common Issues](#common-issues)

---

## Prerequisites

### Required Software

- **Python**: 3.11 or higher
  ```bash
  python --version  # Should show 3.11+
  ```

- **Node.js**: 20 or higher
  ```bash
  node --version  # Should show v20+
  npm --version   # Should show 10+
  ```

- **Git**: For version control
  ```bash
  git --version
  ```

### Neon PostgreSQL Account

1. Sign up at [neon.tech](https://neon.tech)
2. Create a new project
3. Copy the connection string (looks like: `postgresql://user:password@ep-xxx.neon.tech/dbname?sslmode=require`)
4. Keep this connection string handy for backend setup

---

## Backend Setup

### 1. Navigate to Backend Directory

```bash
cd backend
```

### 2. Create Virtual Environment

**On macOS/Linux**:
```bash
python -m venv venv
source venv/bin/activate
```

**On Windows**:
```bash
python -m venv venv
venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

**Expected dependencies** (see `requirements.txt`):
```
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlmodel==0.0.14
psycopg2-binary==2.9.9
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.6
pydantic[email]==2.5.0
```

### 4. Configure Environment Variables

Create `.env` file in `backend/` directory:

```bash
cp .env.example .env
```

Edit `.env` with your values:

```env
# Database
DATABASE_URL=postgresql://user:password@ep-xxx.neon.tech/dbname?sslmode=require

# JWT
JWT_SECRET_KEY=your-secret-key-min-256-bits-generate-with-openssl
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_HOURS=24

# CORS
CORS_ORIGINS=http://localhost:3000,http://localhost:3001

# Environment
ENVIRONMENT=development
```

**Generate secure JWT secret**:
```bash
openssl rand -hex 32
```

### 5. Initialize Database

The application will automatically create tables on startup, but you can verify:

```bash
python -c "from app.database import create_db_and_tables; create_db_and_tables()"
```

### 6. Run Backend Server

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

**Expected output**:
```
INFO:     Will watch for changes in these directories: ['/path/to/backend']
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
INFO:     Started reloader process
INFO:     Started server process
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

**Verify backend is running**:
- Open browser: http://localhost:8000/docs
- You should see Swagger UI with API documentation

---

## Frontend Setup

### 1. Navigate to Frontend Directory

**Open a new terminal** (keep backend running) and navigate to frontend:

```bash
cd frontend
```

### 2. Install Dependencies

```bash
npm install
```

**Expected dependencies** (see `package.json`):
```json
{
  "dependencies": {
    "next": "^16.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "typescript": "^5.3.0"
  },
  "devDependencies": {
    "@types/node": "^20.10.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "autoprefixer": "^10.4.16",
    "postcss": "^8.4.32",
    "tailwindcss": "^3.4.0",
    "eslint": "^8.55.0",
    "eslint-config-next": "^16.0.0"
  }
}
```

### 3. Configure Environment Variables

Create `.env.local` file in `frontend/` directory:

```bash
cp .env.example .env.local
```

Edit `.env.local`:

```env
NEXT_PUBLIC_API_URL=http://localhost:8000/api
```

### 4. Run Frontend Development Server

```bash
npm run dev
```

**Expected output**:
```
  ▲ Next.js 16.0.0
  - Local:        http://localhost:3000
  - Network:      http://192.168.1.x:3000

 ✓ Ready in 2.5s
```

**Verify frontend is running**:
- Open browser: http://localhost:3000
- You should see the landing page with "Register" and "Login" links

---

## Database Setup

### Option 1: Automatic (Recommended)

The backend automatically creates tables on startup via SQLModel:

```python
# In app/main.py
@app.on_event("startup")
def on_startup():
    create_db_and_tables()
```

Just start the backend and tables will be created.

### Option 2: Manual Migration (Using Alembic)

**Initialize Alembic** (first time only):

```bash
cd backend
alembic init alembic
```

**Configure `alembic.ini`**:
```ini
sqlalchemy.url = ${DATABASE_URL}
```

**Generate initial migration**:
```bash
alembic revision --autogenerate -m "Initial schema"
```

**Apply migration**:
```bash
alembic upgrade head
```

### Verify Database Tables

**Connect to Neon database** via their web console or psql:

```sql
-- List tables
\dt

-- Expected tables:
-- users
-- tasks

-- Verify users table
SELECT * FROM users;

-- Verify tasks table
SELECT * FROM tasks;
```

---

## Running the Application

### Start Both Servers

**Terminal 1 (Backend)**:
```bash
cd backend
source venv/bin/activate  # On Windows: venv\Scripts\activate
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

**Terminal 2 (Frontend)**:
```bash
cd frontend
npm run dev
```

### Access the Application

- **Frontend**: http://localhost:3000
- **Backend API Docs**: http://localhost:8000/docs
- **Backend ReDoc**: http://localhost:8000/redoc

### Complete User Journey

1. **Register Account**:
   - Navigate to http://localhost:3000
   - Click "Register" link
   - Enter email: `test@example.com`
   - Enter password: `SecurePass123!`
   - Click "Register"
   - Should redirect to dashboard

2. **Create First Task**:
   - On dashboard, click "Add Task" button
   - Enter title: `Buy groceries`
   - Enter description: `Milk, eggs, bread`
   - Select priority: `High`
   - Click "Create"
   - Task should appear in list

3. **Update Task**:
   - Click "Edit" button on task
   - Change title to: `Buy organic groceries`
   - Click "Save"
   - Changes should reflect immediately

4. **Toggle Completion**:
   - Click checkbox next to task
   - Task should show as complete (strikethrough, checkmark)
   - Click again to mark incomplete

5. **Delete Task**:
   - Click "Delete" button
   - Confirm deletion in modal
   - Task should disappear

6. **Logout**:
   - Click "Logout" button in navbar
   - Should redirect to login page
   - Protected routes should be inaccessible

7. **Login Again**:
   - Click "Login" link
   - Enter email: `test@example.com`
   - Enter password: `SecurePass123!`
   - Should redirect to dashboard
   - Previous tasks should persist (database storage)

---

## Testing the API

### Using Swagger UI (Recommended)

1. Open http://localhost:8000/docs
2. Click "Authorize" button (if using Bearer token approach)
   - Or use Swagger's cookie support
3. Test endpoints interactively

### Using cURL

**Register User**:
```bash
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"SecurePass123!"}' \
  -c cookies.txt \
  -v
```

**Login**:
```bash
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"SecurePass123!"}' \
  -c cookies.txt \
  -v
```

**Get Tasks**:
```bash
curl -X GET http://localhost:8000/api/tasks \
  -b cookies.txt
```

**Create Task**:
```bash
curl -X POST http://localhost:8000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Buy groceries","description":"Milk, eggs","priority":"high"}' \
  -b cookies.txt
```

**Update Task**:
```bash
curl -X PUT http://localhost:8000/api/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"title":"Buy organic groceries"}' \
  -b cookies.txt
```

**Toggle Task**:
```bash
curl -X PATCH http://localhost:8000/api/tasks/1/toggle \
  -b cookies.txt
```

**Delete Task**:
```bash
curl -X DELETE http://localhost:8000/api/tasks/1 \
  -b cookies.txt
```

**Logout**:
```bash
curl -X POST http://localhost:8000/api/auth/logout \
  -b cookies.txt
```

### Using Postman

1. Import OpenAPI spec from http://localhost:8000/openapi.json
2. Configure environment variables:
   - `baseUrl`: `http://localhost:8000/api`
3. Enable "Automatically follow redirects" in settings
4. Enable "Send cookies" in request settings
5. Test endpoints using collections

---

## Common Issues

### Backend Issues

**Issue**: `ModuleNotFoundError: No module named 'fastapi'`

**Solution**: Activate virtual environment and install dependencies:
```bash
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

---

**Issue**: `sqlalchemy.exc.OperationalError: could not connect to server`

**Solution**: Verify Neon connection string in `.env`:
- Ensure DATABASE_URL is correct
- Check Neon project is active
- Verify network connectivity
```bash
# Test connection
python -c "from app.database import engine; print(engine.connect())"
```

---

**Issue**: `jose.exceptions.JWTError: Invalid token`

**Solution**: Generate new JWT secret key:
```bash
openssl rand -hex 32
```
Update `.env` file with new `JWT_SECRET_KEY`.

---

**Issue**: CORS error in browser console

**Solution**: Verify CORS_ORIGINS in `.env`:
```env
CORS_ORIGINS=http://localhost:3000
```
Restart backend server after changes.

---

### Frontend Issues

**Issue**: `Error: EADDRINUSE: address already in use :::3000`

**Solution**: Kill process on port 3000 or use different port:
```bash
# Kill existing process
lsof -ti:3000 | xargs kill -9

# Or use different port
npm run dev -- -p 3001
```

---

**Issue**: `Error: Failed to fetch` or `NetworkError` when calling API

**Solution**:
1. Verify backend is running on port 8000
2. Check NEXT_PUBLIC_API_URL in `.env.local`:
   ```env
   NEXT_PUBLIC_API_URL=http://localhost:8000/api
   ```
3. Restart frontend server after changes

---

**Issue**: Cookies not being sent to backend

**Solution**:
1. Ensure `credentials: 'include'` in fetch calls
2. Verify CORS allows credentials in backend
3. Check cookies in browser DevTools → Application → Cookies
4. Ensure SameSite attribute is compatible

---

**Issue**: Middleware redirecting incorrectly

**Solution**: Clear browser cookies:
- Open DevTools → Application → Cookies
- Delete `access_token` cookie
- Refresh page

---

### Database Issues

**Issue**: `sqlalchemy.exc.ProgrammingError: relation "users" does not exist`

**Solution**: Create database tables:
```bash
cd backend
python -c "from app.database import create_db_and_tables; create_db_and_tables()"
```

---

**Issue**: Duplicate email error not showing correct message

**Solution**: Verify API error handling in backend:
```python
# In app/routers/auth.py
except IntegrityError:
    raise HTTPException(status_code=409, detail="Email already registered")
```

---

## Development Tips

### Hot Reload

Both backend and frontend support hot reload:
- **Backend**: uvicorn `--reload` flag watches for file changes
- **Frontend**: Next.js dev server automatically reloads

### Logging

**Backend logs**:
```bash
# In app/main.py
import logging
logging.basicConfig(level=logging.INFO)
```

**View SQL queries**:
```python
# In app/database.py
engine = create_engine(DATABASE_URL, echo=True)  # Set echo=True
```

### Database Inspection

**Using Neon Web Console**:
1. Go to neon.tech dashboard
2. Select your project
3. Click "SQL Editor"
4. Run queries directly

**Using psql**:
```bash
psql "postgresql://user:password@ep-xxx.neon.tech/dbname?sslmode=require"
```

### API Documentation

FastAPI automatically generates interactive docs:
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc
- OpenAPI JSON: http://localhost:8000/openapi.json

---

## Next Steps

Once the application is running:

1. ✅ Verify all API endpoints work via Swagger UI
2. ✅ Test full user journey (register → login → CRUD → logout)
3. ✅ Verify user isolation (create two users, ensure tasks are separate)
4. ✅ Test error handling (invalid inputs, unauthorized access)
5. ✅ Check browser console for errors
6. ✅ Verify JWT cookies in browser DevTools

---

## Production Deployment (Future)

### Backend Deployment

**Recommended platforms**:
- Railway
- Render
- Fly.io

**Environment variables to configure**:
- DATABASE_URL (Neon production URL)
- JWT_SECRET_KEY (new secret for production)
- CORS_ORIGINS (production frontend URL)
- ENVIRONMENT=production

**Additional steps**:
- Set `secure=True` in cookie settings (HTTPS only)
- Use Alembic for database migrations
- Configure logging and monitoring

### Frontend Deployment

**Recommended platform**: Vercel

**Environment variables**:
- NEXT_PUBLIC_API_URL (production backend URL)

**Build command**:
```bash
npm run build
```

**Start command**:
```bash
npm start
```

### Database

Neon is already serverless - just use production connection string.

---

## Support

For issues or questions:
1. Check this quickstart guide
2. Review API documentation at /docs
3. Consult research.md for architecture decisions
4. Review data-model.md for database schema
5. Check contracts/api-spec.md for API contracts
