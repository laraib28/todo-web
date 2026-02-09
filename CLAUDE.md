# Todo Web App - Phase 2

Full-stack web application with FastAPI backend, Next.js 16 frontend, PostgreSQL database, and JWT authentication.

## Quick Start

### Backend
```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload
```

### Frontend
```bash
cd frontend
npm install
npm run dev
```

## Architecture
- **Backend**: FastAPI + SQLAlchemy + PostgreSQL + JWT auth
- **Frontend**: Next.js 16 + TypeScript + Tailwind CSS
- **Auth**: JWT tokens with bcrypt password hashing
- **Database**: PostgreSQL with Alembic migrations

## API Endpoints
- `POST /api/auth/register` - Register user
- `POST /api/auth/login` - Login, get JWT
- `GET /api/tasks` - List tasks
- `POST /api/tasks` - Create task
- `PUT /api/tasks/{id}` - Update task
- `DELETE /api/tasks/{id}` - Delete task

## Environment Variables
Copy `.env.example` to `.env` and configure:
- `DATABASE_URL` - PostgreSQL connection string
- `SECRET_KEY` - JWT secret key
