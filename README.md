# Todo Web App - Phase 2

A full-stack Todo web application with FastAPI backend, Next.js 16 frontend, PostgreSQL, and JWT authentication.

## Features
- User registration and login with JWT auth
- Create, read, update, delete todos
- Priority levels and status tracking
- Responsive UI with Tailwind CSS
- PostgreSQL database with Alembic migrations

## Tech Stack
- **Backend**: Python, FastAPI, SQLAlchemy, PostgreSQL
- **Frontend**: Next.js 16, TypeScript, Tailwind CSS
- **Auth**: JWT + bcrypt
- **Database**: PostgreSQL + Alembic migrations

## Quick Start

### Backend
```bash
cd backend
pip install -r requirements.txt
# Configure .env with DATABASE_URL and SECRET_KEY
uvicorn app.main:app --reload
```

### Frontend
```bash
cd frontend
npm install
npm run dev
```

## Project Structure
```
backend/
├── app/
│   ├── main.py         # FastAPI app
│   ├── models.py       # SQLAlchemy models
│   ├── schemas.py      # Pydantic schemas
│   ├── database.py     # DB connection
│   ├── auth.py         # JWT auth
│   ├── dependencies.py # DI dependencies
│   └── routers/
│       ├── auth.py     # Auth endpoints
│       └── tasks.py    # Task CRUD
├── alembic/            # DB migrations
├── Dockerfile
└── requirements.txt
frontend/
├── app/               # Next.js pages
├── components/        # React components
├── lib/              # API client
├── Dockerfile
└── package.json
```

## Built with Claude Code + SpecKit
