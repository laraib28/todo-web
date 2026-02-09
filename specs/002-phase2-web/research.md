# Research: Phase-II Full-Stack Web Todo Application

**Feature**: 002-phase2-web
**Date**: 2026-01-01
**Purpose**: Technology selection, architecture patterns, and integration strategies for multi-user web todo application

## Research Questions

### 1. Next.js 16+ App Router Architecture
### 2. FastAPI + SQLModel Integration Patterns
### 3. Better Auth JWT Implementation
### 4. Neon Serverless PostgreSQL Connection Strategy
### 5. Frontend-Backend Communication Security
### 6. User Isolation and Multi-Tenancy Patterns

---

## 1. Next.js 16+ App Router Architecture

### Decision
Use Next.js 16+ with App Router (app/ directory) for frontend implementation.

### Rationale
- **Server Components by Default**: Improved performance through automatic server-side rendering
- **Built-in Routing**: File-system based routing with layouts, loading states, error boundaries
- **API Routes Alternative**: Server Actions provide type-safe server communication
- **Middleware Support**: Built-in middleware for JWT authentication and route protection
- **Streaming & Suspense**: Better UX with progressive loading
- **TypeScript First**: Excellent TypeScript support out of the box

### Implementation Approach
```
frontend/
├── app/
│   ├── layout.tsx           # Root layout with Navbar
│   ├── page.tsx             # Landing/redirect page
│   ├── register/
│   │   └── page.tsx         # Registration form (client component)
│   ├── login/
│   │   └── page.tsx         # Login form (client component)
│   └── dashboard/
│       └── page.tsx         # Dashboard with task list (server component)
├── components/
│   ├── TaskList.tsx         # Client component for interactive list
│   ├── TaskItem.tsx         # Client component for task row
│   ├── TaskForm.tsx         # Client component for add/edit modal
│   └── Navbar.tsx           # Client component for navigation
├── lib/
│   ├── api.ts               # API client functions
│   └── auth.ts              # Auth utilities
└── middleware.ts            # Route protection middleware
```

### Key Patterns
1. **Server Components**: Use for data fetching and initial render (dashboard)
2. **Client Components**: Use for interactivity ('use client' for forms, modals)
3. **Server Actions**: Consider for form submissions as alternative to API routes
4. **Middleware**: Protect routes by checking JWT cookie before page load

### Alternatives Considered
- **Pages Router**: Rejected - older pattern, less performant, more boilerplate
- **React SPA + Vite**: Rejected - loses SSR benefits, SEO disadvantages
- **Remix**: Rejected - less ecosystem maturity, different patterns

---

## 2. FastAPI + SQLModel Integration Patterns

### Decision
Use FastAPI with SQLModel for ORM and Pydantic schemas combined.

### Rationale
- **Type Safety**: SQLModel combines SQLAlchemy models + Pydantic schemas in one definition
- **Less Boilerplate**: Single model definition serves both database and API validation
- **Async Support**: FastAPI native async/await for better performance
- **Auto Documentation**: OpenAPI/Swagger docs at /docs endpoint
- **Dependency Injection**: Clean pattern for database sessions and current user
- **Validation**: Pydantic validation built into request/response models

### Implementation Approach

**Database Connection Pattern**:
```python
# backend/app/database.py
from sqlmodel import create_engine, Session, SQLModel

DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL, echo=True)

def get_session():
    with Session(engine) as session:
        yield session

def create_db_and_tables():
    SQLModel.metadata.create_all(engine)
```

**Model Pattern (SQLModel)**:
```python
# backend/app/models.py
from sqlmodel import Field, SQLModel, Relationship
from datetime import datetime

class User(SQLModel, table=True):
    __tablename__ = "users"

    id: int | None = Field(default=None, primary_key=True)
    email: str = Field(unique=True, index=True)
    hashed_password: str
    created_at: datetime = Field(default_factory=datetime.utcnow)

    tasks: list["Task"] = Relationship(back_populates="user")

class Task(SQLModel, table=True):
    __tablename__ = "tasks"

    id: int | None = Field(default=None, primary_key=True)
    user_id: int = Field(foreign_key="users.id", ondelete="CASCADE")
    title: str = Field(max_length=200)
    description: str = Field(default="")
    priority: str = Field(default="medium")  # high/medium/low
    is_complete: bool = Field(default=False)
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    user: User = Relationship(back_populates="tasks")
```

**Schema Pattern (Pydantic for API)**:
```python
# backend/app/schemas.py
from pydantic import BaseModel, EmailStr, Field

class UserRegister(BaseModel):
    email: EmailStr
    password: str = Field(min_length=8)

class UserLogin(BaseModel):
    email: EmailStr
    password: str

class TaskCreate(BaseModel):
    title: str = Field(min_length=1, max_length=200)
    description: str = ""
    priority: str = "medium"

class TaskUpdate(BaseModel):
    title: str | None = None
    description: str | None = None
    priority: str | None = None

class TaskResponse(BaseModel):
    id: int
    title: str
    description: str
    priority: str
    is_complete: bool
    created_at: datetime
    updated_at: datetime
```

**Dependency Injection Pattern**:
```python
# backend/app/dependencies.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer
from sqlmodel import Session
from app.database import get_session
from app.auth import decode_jwt

security = HTTPBearer()

async def get_current_user(
    token: str = Depends(security),
    session: Session = Depends(get_session)
) -> User:
    payload = decode_jwt(token.credentials)
    if not payload:
        raise HTTPException(status_code=401, detail="Invalid token")

    user = session.get(User, payload["user_id"])
    if not user:
        raise HTTPException(status_code=401, detail="User not found")

    return user
```

### Alternatives Considered
- **SQLAlchemy + Pydantic separately**: Rejected - double model definitions, more boilerplate
- **Django ORM**: Rejected - requires Django framework, overkill for API
- **Prisma (Python)**: Rejected - less mature, different migration strategy

---

## 3. Better Auth JWT Implementation

### Decision
Use python-jose for JWT encoding/decoding with passlib for password hashing.

### Rationale
- **Better Auth Context**: The spec mentions "Better Auth" - interpreted as best-practices JWT auth (not a specific library)
- **python-jose**: Industry standard for JWT in Python (used in FastAPI docs)
- **passlib[bcrypt]**: Secure password hashing with bcrypt algorithm
- **httpOnly Cookies**: Prevent XSS attacks by storing JWT in httpOnly cookies
- **Token Expiration**: Built-in exp claim for security

### Implementation Approach

**Password Hashing**:
```python
# backend/app/auth.py
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)
```

**JWT Generation**:
```python
from jose import JWTError, jwt
from datetime import datetime, timedelta
import os

SECRET_KEY = os.getenv("JWT_SECRET_KEY")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_HOURS = 24

def create_jwt(user_id: int) -> str:
    expire = datetime.utcnow() + timedelta(hours=ACCESS_TOKEN_EXPIRE_HOURS)
    payload = {
        "user_id": user_id,
        "exp": expire
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def decode_jwt(token: str) -> dict | None:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        return None
```

**Cookie Handling (Frontend)**:
```typescript
// frontend/lib/api.ts
async function login(email: string, password: string) {
  const response = await fetch('http://localhost:8000/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    credentials: 'include',  // Important: send cookies
    body: JSON.stringify({ email, password })
  });

  // Backend sets httpOnly cookie in response
  return response.json();
}
```

**Cookie Setting (Backend)**:
```python
from fastapi import Response

@router.post("/auth/login")
async def login(user: UserLogin, response: Response, session: Session = Depends(get_session)):
    # Verify credentials...
    token = create_jwt(user.id)

    response.set_cookie(
        key="access_token",
        value=token,
        httponly=True,  # XSS protection
        secure=True,    # HTTPS only (production)
        samesite="lax", # CSRF protection
        max_age=86400   # 24 hours
    )

    return {"message": "Login successful"}
```

### Security Considerations
1. **Secret Key**: Generate strong secret (256+ bits), store in environment variable
2. **httpOnly Cookie**: Prevents JavaScript access (XSS protection)
3. **Secure Flag**: HTTPS only in production
4. **SameSite**: Lax or Strict for CSRF protection
5. **Token Expiration**: 24 hour expiry, require re-login
6. **Password Hashing**: bcrypt with cost factor 12

### Alternatives Considered
- **PyJWT**: Rejected - python-jose has better FastAPI integration
- **OAuth2 Password Flow**: Rejected - overhead for MVP, JWT simpler
- **Session-based Auth**: Rejected - stateful, doesn't scale as well
- **localStorage for JWT**: Rejected - vulnerable to XSS

---

## 4. Neon Serverless PostgreSQL Connection Strategy

### Decision
Use Neon connection string with SQLModel/SQLAlchemy engine, connection pooling enabled.

### Rationale
- **Serverless**: Auto-scaling, no infrastructure management
- **Connection String**: Standard PostgreSQL wire protocol
- **Pooling**: SQLAlchemy built-in connection pooling for efficiency
- **SSL**: Required for Neon, handled automatically

### Implementation Approach

**Environment Configuration**:
```bash
# backend/.env
DATABASE_URL=postgresql://user:password@ep-xxx.neon.tech/dbname?sslmode=require
JWT_SECRET_KEY=your-secret-key-min-256-bits
CORS_ORIGINS=http://localhost:3000,http://localhost:3001
```

**Connection Setup**:
```python
# backend/app/database.py
from sqlmodel import create_engine, Session, SQLModel
import os

DATABASE_URL = os.getenv("DATABASE_URL")

# Connection pooling configuration
engine = create_engine(
    DATABASE_URL,
    echo=True,  # SQL logging for development
    pool_size=5,  # Connection pool size
    max_overflow=10,  # Additional connections when pool exhausted
    pool_pre_ping=True,  # Verify connections before using
)

def get_session():
    """Dependency for database sessions"""
    with Session(engine) as session:
        yield session

def create_db_and_tables():
    """Create all tables on startup"""
    SQLModel.metadata.create_all(engine)
```

**Startup Event**:
```python
# backend/app/main.py
from fastapi import FastAPI
from app.database import create_db_and_tables

app = FastAPI()

@app.on_event("startup")
def on_startup():
    create_db_and_tables()
```

### Connection Best Practices
1. **Pool Pre-Ping**: Verify connection alive before use (handles serverless cold starts)
2. **Pool Size**: 5 connections sufficient for MVP (< 100 concurrent users)
3. **SSL Mode**: Required by Neon (`?sslmode=require` in connection string)
4. **Environment Variables**: Never hardcode credentials
5. **Error Handling**: Catch connection errors, return 503 Service Unavailable

### Alternatives Considered
- **asyncpg**: Rejected - adds complexity, async not required for MVP
- **psycopg2**: Rejected - SQLAlchemy handles driver selection
- **Local PostgreSQL**: Rejected - spec requires Neon Serverless

---

## 5. Frontend-Backend Communication Security

### Decision
CORS configuration + JWT in httpOnly cookies + input validation on both ends.

### Rationale
- **CORS**: Prevent unauthorized origins from accessing API
- **JWT Cookies**: XSS protection through httpOnly flag
- **Double Validation**: Frontend validates for UX, backend validates for security
- **HTTPS**: Required in production for cookie security

### Implementation Approach

**CORS Configuration (Backend)**:
```python
# backend/app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
import os

app = FastAPI()

CORS_ORIGINS = os.getenv("CORS_ORIGINS", "http://localhost:3000").split(",")

app.add_middleware(
    CORSMiddleware,
    allow_origins=CORS_ORIGINS,  # Frontend origin
    allow_credentials=True,  # Allow cookies
    allow_methods=["GET", "POST", "PUT", "DELETE", "PATCH"],
    allow_headers=["Content-Type", "Authorization"],
)
```

**API Client (Frontend)**:
```typescript
// frontend/lib/api.ts
const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:8000/api';

async function apiRequest<T>(
  endpoint: string,
  options: RequestInit = {}
): Promise<T> {
  const response = await fetch(`${API_BASE_URL}${endpoint}`, {
    ...options,
    credentials: 'include',  // Send cookies with every request
    headers: {
      'Content-Type': 'application/json',
      ...options.headers,
    },
  });

  if (response.status === 401) {
    // Redirect to login on unauthorized
    window.location.href = '/login';
    throw new Error('Unauthorized');
  }

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.detail || 'Request failed');
  }

  return response.json();
}

// Task API functions
export const api = {
  // Auth
  register: (email: string, password: string) =>
    apiRequest('/auth/register', {
      method: 'POST',
      body: JSON.stringify({ email, password }),
    }),

  login: (email: string, password: string) =>
    apiRequest('/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password }),
    }),

  logout: () => apiRequest('/auth/logout', { method: 'POST' }),

  // Tasks
  getTasks: () => apiRequest<Task[]>('/tasks'),
  createTask: (task: TaskCreate) =>
    apiRequest('/tasks', {
      method: 'POST',
      body: JSON.stringify(task),
    }),
  updateTask: (id: number, task: TaskUpdate) =>
    apiRequest(`/tasks/${id}`, {
      method: 'PUT',
      body: JSON.stringify(task),
    }),
  toggleTask: (id: number) =>
    apiRequest(`/tasks/${id}/toggle`, { method: 'PATCH' }),
  deleteTask: (id: number) =>
    apiRequest(`/tasks/${id}`, { method: 'DELETE' }),
};
```

**Route Protection Middleware (Frontend)**:
```typescript
// frontend/middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('access_token');
  const isAuthPage = request.nextUrl.pathname.startsWith('/login') ||
                     request.nextUrl.pathname.startsWith('/register');
  const isProtectedPage = request.nextUrl.pathname.startsWith('/dashboard');

  // Redirect authenticated users away from login/register
  if (token && isAuthPage) {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }

  // Redirect unauthenticated users to login
  if (!token && isProtectedPage) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

### Security Headers
```python
# backend/app/main.py
@app.middleware("http")
async def add_security_headers(request, call_next):
    response = await call_next(request)
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"
    return response
```

### Validation Strategy
1. **Frontend Validation**: Immediate feedback, better UX (email format, password length)
2. **Backend Validation**: Security boundary, never trust client input
3. **Sanitization**: Prevent XSS/SQL injection (Pydantic + SQLModel parameterized queries)

---

## 6. User Isolation and Multi-Tenancy Patterns

### Decision
Row-level security through user_id foreign key + automatic filtering in all queries.

### Rationale
- **Database Constraint**: Foreign key ensures referential integrity
- **Automatic Filtering**: Include WHERE user_id = current_user.id in all queries
- **Dependency Injection**: Current user from JWT, injected into endpoint handlers
- **403 Forbidden**: Explicit errors when user tries to access others' data

### Implementation Approach

**Database Model with Foreign Key**:
```python
# backend/app/models.py
class Task(SQLModel, table=True):
    __tablename__ = "tasks"

    id: int | None = Field(default=None, primary_key=True)
    user_id: int = Field(foreign_key="users.id", ondelete="CASCADE")
    # ... other fields
```

**Query Pattern with User Filtering**:
```python
# backend/app/routers/tasks.py
from fastapi import APIRouter, Depends, HTTPException
from sqlmodel import Session, select
from app.models import Task
from app.dependencies import get_current_user, get_session

router = APIRouter(prefix="/tasks", tags=["tasks"])

@router.get("/")
async def get_tasks(
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
) -> list[Task]:
    """Get all tasks for current user (user isolation enforced)"""
    statement = select(Task).where(Task.user_id == current_user.id)
    tasks = session.exec(statement).all()
    return tasks

@router.post("/")
async def create_task(
    task_data: TaskCreate,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
) -> Task:
    """Create task for current user"""
    task = Task(
        **task_data.dict(),
        user_id=current_user.id  # Automatically set to current user
    )
    session.add(task)
    session.commit()
    session.refresh(task)
    return task

@router.put("/{task_id}")
async def update_task(
    task_id: int,
    task_data: TaskUpdate,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
) -> Task:
    """Update task (with ownership check)"""
    task = session.get(Task, task_id)

    if not task:
        raise HTTPException(status_code=404, detail="Task not found")

    if task.user_id != current_user.id:
        raise HTTPException(status_code=403, detail="Not authorized")

    # Update fields
    for key, value in task_data.dict(exclude_unset=True).items():
        setattr(task, key, value)

    task.updated_at = datetime.utcnow()
    session.add(task)
    session.commit()
    session.refresh(task)
    return task
```

**User Isolation Checklist**:
- ✅ Foreign key constraint on tasks.user_id
- ✅ CASCADE delete (deleting user deletes their tasks)
- ✅ All SELECT queries filter by user_id
- ✅ All INSERT operations set user_id from JWT
- ✅ All UPDATE/DELETE operations verify ownership (403 if mismatch)
- ✅ JWT validates user exists before accessing data
- ✅ No task IDs exposed across users (queries are scoped)

### Testing Strategy
1. Create User A and User B
2. User A creates Task 1
3. User B tries to access Task 1 by ID → 403 Forbidden
4. User B creates Task 2
5. User A lists tasks → only Task 1 visible
6. User B lists tasks → only Task 2 visible

### Edge Cases
- **JWT with non-existent user_id**: 401 Unauthorized (user not found)
- **Direct task access by ID**: 403 Forbidden if user_id mismatch
- **Bulk operations**: Apply same user_id filter
- **User deletion**: CASCADE deletes all user's tasks

---

## Technology Stack Summary

### Frontend
- **Framework**: Next.js 16+ (App Router)
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS
- **HTTP Client**: fetch API with wrapper
- **Auth**: httpOnly cookies + middleware

### Backend
- **Framework**: FastAPI 0.104+
- **Language**: Python 3.11+
- **ORM**: SQLModel 0.0.14+
- **Auth**: python-jose[cryptography] + passlib[bcrypt]
- **Database Driver**: psycopg2 (via SQLAlchemy)
- **Validation**: Pydantic (built into FastAPI/SQLModel)

### Database
- **Provider**: Neon Serverless PostgreSQL
- **Connection**: Connection string with SSL
- **Pooling**: SQLAlchemy pool (size=5, max_overflow=10)
- **Migrations**: Alembic (for schema versioning)

### Development Tools
- **API Testing**: FastAPI /docs (Swagger UI)
- **Type Checking**: mypy (backend), TypeScript compiler (frontend)
- **Linting**: ruff (backend), ESLint (frontend)
- **Formatting**: black (backend), Prettier (frontend)

### Deployment (Future)
- Frontend: Vercel
- Backend: Railway / Render / Fly.io
- Database: Neon (already serverless)

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                         Browser                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Next.js Frontend (Port 3000)                        │   │
│  │  - App Router (app/)                                 │   │
│  │  - TypeScript + Tailwind                            │   │
│  │  - Client Components (forms, modals)                │   │
│  │  - Server Components (dashboard)                    │   │
│  └──────────────────┬───────────────────────────────────┘   │
│                     │                                        │
│                     │ HTTP + JWT Cookie (httpOnly)           │
│                     │                                        │
└─────────────────────┼────────────────────────────────────────┘
                      │
                      ▼
         ┌────────────────────────────┐
         │  CORS Middleware           │
         └────────────┬───────────────┘
                      │
         ┌────────────▼───────────────┐
         │  FastAPI Backend (8000)    │
         │  ┌──────────────────────┐  │
         │  │  Auth Router         │  │
         │  │  - /register         │  │
         │  │  - /login            │  │
         │  │  - /logout           │  │
         │  └──────────────────────┘  │
         │  ┌──────────────────────┐  │
         │  │  JWT Middleware      │  │
         │  │  (Decode & Validate) │  │
         │  └──────────┬───────────┘  │
         │             │               │
         │  ┌──────────▼───────────┐  │
         │  │  Tasks Router        │  │
         │  │  - GET /tasks        │  │
         │  │  - POST /tasks       │  │
         │  │  - PUT /tasks/{id}   │  │
         │  │  - PATCH /{id}/toggle│  │
         │  │  - DELETE /tasks/{id}│  │
         │  └──────────┬───────────┘  │
         │             │               │
         │  ┌──────────▼───────────┐  │
         │  │  SQLModel (ORM)      │  │
         │  │  - User model        │  │
         │  │  - Task model        │  │
         │  └──────────┬───────────┘  │
         └─────────────┼───────────────┘
                       │
                       │ SQL Queries (Parameterized)
                       │
         ┌─────────────▼───────────────┐
         │  Neon PostgreSQL             │
         │  ┌────────┐    ┌──────────┐ │
         │  │ users  │    │  tasks   │ │
         │  │        │◄───│          │ │
         │  │ id (PK)│    │ user_id  │ │
         │  │ email  │    │ (FK)     │ │
         │  └────────┘    └──────────┘ │
         └──────────────────────────────┘
```

---

## Risk Analysis

### Technical Risks

1. **JWT in Cookies vs Authorization Header**
   - Risk: Some guides use Authorization header
   - Mitigation: httpOnly cookies more secure (XSS protection)
   - Decision: Use cookies as specified in research

2. **Next.js Middleware JWT Validation**
   - Risk: Middleware can't call external APIs easily
   - Mitigation: Validate JWT signature in middleware, full validation in API
   - Decision: Client-side check in middleware, server-side enforcement in API

3. **Neon Connection Cold Starts**
   - Risk: Serverless databases may have latency on first connection
   - Mitigation: pool_pre_ping=True, connection pooling
   - Decision: Accept slight cold start delay for serverless benefits

4. **CORS Preflight Requests**
   - Risk: OPTIONS requests may not include cookies
   - Mitigation: Allow OPTIONS in CORS config, don't require auth for preflight
   - Decision: FastAPI CORS middleware handles this automatically

### Security Risks

1. **Password Storage**
   - Risk: Plain text passwords catastrophic
   - Mitigation: bcrypt hashing with cost factor 12
   - Decision: passlib with bcrypt (industry standard)

2. **SQL Injection**
   - Risk: User input in SQL queries
   - Mitigation: SQLModel uses parameterized queries
   - Decision: Never use string interpolation for SQL

3. **XSS Attacks**
   - Risk: Malicious scripts in task titles/descriptions
   - Mitigation: React auto-escapes, backend validates
   - Decision: Trust React escaping, validate max lengths

4. **User Enumeration**
   - Risk: Different errors reveal if email exists
   - Mitigation: Same error message for wrong email vs wrong password
   - Decision: "Invalid credentials" for both cases

---

---

## 7. OpenAI Agents SDK - Todo Chatbot Integration

### Decision
Use OpenAI Agents SDK for Python (openai-agents package, v0.6.4+) with function tools to enable intelligent todo management through conversational AI.

### Rationale
- **Production-Ready**: Released March 2025, evolved from experimental Swarm project
- **Lightweight Framework**: Minimal abstractions, few dependencies
- **Function Tools**: Automatic schema generation from Python functions using decorators
- **Built-in Sessions**: Automatic conversation history management (SQLiteSession for MVP)
- **Provider-Agnostic**: Works with OpenAI + 100+ other LLM providers (future flexibility)
- **No Streaming Required**: SDK supports complete responses (non-streaming), simpler for MVP
- **Error Handling**: Native error types for rate limits, API failures, timeouts

### Current SDK Specifications

**Package**: `openai-agents` (PyPI)
**Latest Version**: 0.6.4
**Python Support**: 3.9+
**OpenAI SDK Version**: Requires openai v2.x (v1.x no longer supported)

**Core Dependencies**:
- openai >= 2.0.0
- pydantic >= 2.0
- pydantic-ai (for advanced agent capabilities)
- litellm (for multi-provider LLM support)

**Installation**:
```bash
pip install openai-agents
# or with uv
uv add openai-agents
```

Optional extensions:
```bash
pip install openai-agents[voice]   # Audio support
pip install openai-agents[redis]   # Distributed session storage
```

### Architecture: Agent Setup with Tool Use

**Basic Agent Configuration**:
```python
from openai_agents import Agent, function_tool
from openai import OpenAI

# Initialize OpenAI client
client = OpenAI(api_key="your-api-key")

# Define custom tools as functions
@function_tool
async def create_task(title: str, description: str = "", priority: str = "medium") -> dict:
    """Create a new todo task.

    Args:
        title: Task title (required)
        description: Task description (optional)
        priority: Priority level - high/medium/low
    """
    # Call your database via SQLModel session
    task = Task(
        title=title,
        description=description,
        priority=priority,
        user_id=current_user_id  # Injected from context
    )
    session.add(task)
    session.commit()
    session.refresh(task)
    return {
        "id": task.id,
        "title": task.title,
        "status": "created"
    }

@function_tool
async def get_tasks(priority_filter: str = None) -> list[dict]:
    """Retrieve all user tasks, optionally filtered by priority.

    Args:
        priority_filter: Optional filter for high/medium/low priority
    """
    query = select(Task).where(Task.user_id == current_user_id)
    if priority_filter:
        query = query.where(Task.priority == priority_filter)
    tasks = session.exec(query).all()
    return [{"id": t.id, "title": t.title, "priority": t.priority} for t in tasks]

@function_tool
async def update_task(task_id: int, title: str = None, priority: str = None) -> dict:
    """Update an existing task.

    Args:
        task_id: ID of task to update
        title: New title (optional)
        priority: New priority level (optional)
    """
    task = session.get(Task, task_id)
    if not task or task.user_id != current_user_id:
        return {"error": "Task not found or unauthorized"}

    if title:
        task.title = title
    if priority:
        task.priority = priority

    session.add(task)
    session.commit()
    session.refresh(task)
    return {"id": task.id, "updated": True}

@function_tool
async def toggle_task_complete(task_id: int) -> dict:
    """Toggle a task's completion status.

    Args:
        task_id: ID of task to toggle
    """
    task = session.get(Task, task_id)
    if not task or task.user_id != current_user_id:
        return {"error": "Task not found or unauthorized"}

    task.is_complete = not task.is_complete
    session.add(task)
    session.commit()
    return {"id": task.id, "is_complete": task.is_complete}

@function_tool
async def delete_task(task_id: int) -> dict:
    """Delete a task.

    Args:
        task_id: ID of task to delete
    """
    task = session.get(Task, task_id)
    if not task or task.user_id != current_user_id:
        return {"error": "Task not found or unauthorized"}

    session.delete(task)
    session.commit()
    return {"id": task_id, "deleted": True}

# Create agent with tools
todo_agent = Agent(
    name="Todo Assistant",
    instructions="""You are a helpful todo list assistant. Help users:
- Create, read, update, and delete tasks
- Filter tasks by priority
- Mark tasks as complete
- Provide task summaries

Always confirm actions and be concise. Ask clarifying questions if needed.""",
    model="gpt-4-turbo",  # or "gpt-4o" for faster responses
    tools=[
        create_task,
        get_tasks,
        update_task,
        toggle_task_complete,
        delete_task
    ]
)
```

### Tool Registration Patterns

**Automatic Tool Discovery**:
```python
# Tool name = function name
# Description = docstring
# Parameters = type-annotated function arguments
# Supports: sync, async, type hints, Pydantic models

@function_tool
async def my_function(arg1: str, arg2: int = 5) -> str:
    """Description extracted from docstring."""
    return f"result"
```

**Advanced Configuration**:
```python
from openai_agents import FunctionTool
from pydantic import BaseModel

class TaskInput(BaseModel):
    title: str
    priority: str

class TaskOutput(BaseModel):
    id: int
    created_at: str

# Manual tool creation for complex scenarios
async def run_create_task(ctx, args_json: str) -> str:
    args = TaskInput.model_validate_json(args_json)
    # Your business logic
    return TaskOutput(...).model_dump_json()

custom_tool = FunctionTool(
    name="create_todo",
    description="Create a new todo task with title and priority",
    params_json_schema=TaskInput.model_json_schema(),
    on_invoke_tool=run_create_task
)

agent = Agent(
    name="Assistant",
    tools=[custom_tool]
)
```

**Tool Behavior Control**:
```python
from openai_agents import ModelSettings

# Force tool use / Require specific tool
agent = Agent(
    name="Assistant",
    tools=[create_task, get_tasks],
    model_settings=ModelSettings(
        tool_choice="required"  # Must use a tool
        # or: tool_choice="create_task"  # Must use specific tool
        # or: tool_choice="auto"  # Default (model decides)
        # or: tool_choice="none"  # Disable tools
    )
)

# Control how LLM processes tool results
agent = Agent(
    name="Assistant",
    tools=[create_task],
    tool_use_behavior="run_llm_again"  # Default: LLM processes results
    # or: tool_use_behavior="stop_on_first_tool"  # Return first tool output directly
)
```

**Agents as Tools (Handoffs)**:
```python
# Create specialized agents
query_agent = Agent(
    name="Query Agent",
    instructions="Answer questions about existing tasks",
    tools=[get_tasks]
)

create_agent = Agent(
    name="Create Agent",
    instructions="Create and modify tasks",
    tools=[create_task, update_task]
)

# Orchestrator agent that delegates
main_agent = Agent(
    name="Todo Assistant",
    instructions="Delegate to specialized agents based on user intent",
    tools=[
        query_agent.as_tool(
            tool_name="query_tasks",
            tool_description="Ask about existing tasks"
        ),
        create_agent.as_tool(
            tool_name="manage_tasks",
            tool_description="Create or modify tasks"
        )
    ]
)
```

### Message History & Conversation Context

**Session-Based Automatic History** (Recommended for MVP):
```python
from openai_agents import Agent, Runner, SQLiteSession

async def handle_user_message(user_id: str, message: str):
    """Process a single user message with full conversation history."""

    agent = Agent(
        name="Todo Assistant",
        instructions="Help manage todo tasks",
        tools=[get_tasks, create_task, update_task]
    )

    # Create or load session (automatically persists history)
    session = SQLiteSession(f"user_{user_id}", db_path="./conversations.db")

    # Run agent - automatically includes full conversation history
    result = await Runner.run(
        agent,
        message,  # Current user message
        session=session
    )

    # result.final_output contains agent's response
    # result.messages contains full turn history
    return result.final_output

# Example usage in FastAPI endpoint
@router.post("/chat")
async def chat(
    request: ChatRequest,  # {"message": "Create a task called..."}
    current_user: User = Depends(get_current_user)
) -> ChatResponse:
    response = await handle_user_message(
        user_id=str(current_user.id),
        message=request.message
    )
    return ChatResponse(message=response)
```

**Session Types Available**:
```python
# Option 1: SQLiteSession (default, lightweight)
session = SQLiteSession("session_123", db_path="./conversations.db")

# Option 2: In-memory (testing only)
session = SQLiteSession("session_123", db_path=":memory:")

# Option 3: OpenAI-hosted Conversations API
from openai_agents import OpenAIConversationsSession
session = OpenAIConversationsSession("session_123")

# Option 4: SQLAlchemy (production, any database)
from openai_agents import SQLAlchemySession
from sqlalchemy import create_engine
engine = create_engine("postgresql://...")
session = SQLAlchemySession("session_123", engine=engine, table_name="agent_conversations")

# Option 5: Encrypted (sensitive conversations)
from openai_agents import EncryptedSession
encrypted_session = EncryptedSession(
    SQLiteSession("session_123"),
    encryption_key="your-256-bit-key"
)
```

**Session Operations**:
```python
# Get conversation history
items = session.get_items()  # Returns all conversation items

# Manually add items (advanced)
await session.add_items([...])

# Remove last item (useful for corrections)
last_item = session.pop_item()

# Clear entire session
session.clear_session()

# Context injection (non-LLM accessible)
class RunContextWrapper:
    def __init__(self):
        self.user_id = None
        self.db_session = None

    # Access in tools
    @function_tool
    async def create_task(ctx: RunContextWrapper, title: str):
        # ctx.user_id and ctx.db_session available
        pass
```

**Manual History Management** (If not using Sessions):
```python
from openai_agents import Runner
from openai_agents.types import MessageParam

async def multi_turn_conversation():
    agent = Agent(
        name="Assistant",
        tools=[get_tasks, create_task]
    )

    # Track conversation manually
    messages: list[MessageParam] = []

    # Turn 1
    user_message_1 = "Show me high priority tasks"
    result_1 = await Runner.run(agent, user_message_1, messages=messages)
    print(result_1.final_output)

    # Append conversation history
    messages.extend(result_1.messages)

    # Turn 2 (includes context from Turn 1)
    user_message_2 = "Create a task for the highest priority one"
    result_2 = await Runner.run(agent, user_message_2, messages=messages)
    print(result_2.final_output)

    # Continue conversation
    messages.extend(result_2.messages)
```

### Error Handling & Resilience

**Error Types & Handling**:
```python
from openai import (
    RateLimitError,
    APIConnectionError,
    APIError,
    APITimeoutError,
    APIStatusError
)
import time
from typing import TypeVar, Callable, Any

T = TypeVar('T')

async def run_with_retry(
    async_fn: Callable[[], T],
    max_retries: int = 3,
    base_wait: float = 1.0
) -> T:
    """Retry with exponential backoff."""
    for attempt in range(max_retries):
        try:
            return await async_fn()
        except RateLimitError as e:
            wait_time = base_wait * (2 ** attempt) + random.uniform(0, 1)
            print(f"Rate limited. Retrying in {wait_time:.1f}s...")
            time.sleep(wait_time)
        except APITimeoutError:
            if attempt == max_retries - 1:
                raise
            wait_time = base_wait * (2 ** attempt)
            print(f"Timeout. Retrying in {wait_time:.1f}s...")
            time.sleep(wait_time)
        except APIConnectionError as e:
            if attempt == max_retries - 1:
                raise
            wait_time = base_wait * (2 ** attempt)
            print(f"Connection error: {e}. Retrying...")
            time.sleep(wait_time)

# Usage in FastAPI endpoint
@router.post("/chat")
async def chat(request: ChatRequest, current_user: User = Depends(get_current_user)):
    try:
        response = await run_with_retry(
            lambda: handle_user_message(str(current_user.id), request.message)
        )
        return ChatResponse(message=response)
    except RateLimitError:
        raise HTTPException(status_code=429, detail="Service temporarily busy")
    except APIError as e:
        print(f"OpenAI API error: {e}")
        raise HTTPException(status_code=502, detail="AI service error")
    except APITimeoutError:
        raise HTTPException(status_code=504, detail="AI service timeout")
```

**Comprehensive Error Handler Middleware**:
```python
from tenacity import retry, wait_exponential, stop_after_attempt
from openai import RateLimitError

@retry(
    wait=wait_exponential(multiplier=1, min=1, max=60),
    stop=stop_after_attempt(6),
    reraise=True
)
async def agent_run_with_retry(agent, message, session):
    """Automatic retry with exponential backoff (requires tenacity)."""
    result = await Runner.run(agent, message, session=session)
    return result

# Install: pip install tenacity
```

**FastAPI Global Exception Handler**:
```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(RateLimitError)
async def rate_limit_handler(request, exc):
    return JSONResponse(
        status_code=429,
        content={"detail": "Too many requests. Please try again later."}
    )

@app.exception_handler(APITimeoutError)
async def timeout_handler(request, exc):
    return JSONResponse(
        status_code=504,
        content={"detail": "AI service is taking too long. Please try again."}
    )

@app.exception_handler(APIError)
async def api_error_handler(request, exc):
    return JSONResponse(
        status_code=502,
        content={"detail": "AI service error. Please try again later."}
    )
```

**Error Response Taxonomy**:
| Status | Error Type | Recovery |
|--------|-----------|----------|
| 429 | RateLimitError | Retry with exponential backoff |
| 504 | APITimeoutError | Retry, or return cached response |
| 502/500 | APIError | Retry, may need manual investigation |
| 400 | Invalid request | Fix input, don't retry |
| 401 | Invalid API key | Fix credentials in environment |

### Streaming vs Complete Responses

**No Streaming Required for MVP** (Simpler Implementation):
```python
# Default behavior: complete response
async def chat_complete(user_id: str, message: str) -> str:
    """Get complete response from agent."""
    session = SQLiteSession(f"user_{user_id}")
    result = await Runner.run(todo_agent, message, session=session)

    # result.final_output = complete string response
    return result.final_output

# In FastAPI endpoint
@router.post("/chat")
async def chat(request: ChatRequest):
    response = await chat_complete(
        user_id=current_user.id,
        message=request.message
    )
    return ChatResponse(message=response)
```

**Optional: Streaming for Real-Time Updates** (If UX improves):
```python
from openai_agents import Runner

async def chat_streaming(user_id: str, message: str):
    """Stream agent response as it's generated."""
    session = SQLiteSession(f"user_{user_id}")

    # Set stream=True
    async with Runner.stream(
        todo_agent,
        message,
        session=session
    ) as stream:
        async for event in stream:
            # Emit Server-Sent Event (SSE) to frontend
            yield f"data: {json.dumps(event)}\n\n"

# FastAPI endpoint with StreamingResponse
from fastapi.responses import StreamingResponse

@router.post("/chat/stream")
async def chat_stream(request: ChatRequest):
    return StreamingResponse(
        chat_streaming(current_user.id, request.message),
        media_type="text/event-stream"
    )

# Frontend (using EventSource)
const eventSource = new EventSource('/api/chat/stream');
eventSource.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log("Agent:", data.output);
};
```

**Design Decision**: Use complete responses for MVP (faster time-to-market). Add streaming later if UX testing shows user value.

### Integration with SQLModel & FastAPI

**Complete Example: Todo Chatbot Endpoint**:
```python
# backend/app/routers/chat.py
from fastapi import APIRouter, Depends, HTTPException
from sqlmodel import Session
from openai_agents import Agent, Runner, SQLiteSession, function_tool
from openai import APIError
from app.dependencies import get_current_user, get_session
from app.models import User, Task
from pydantic import BaseModel

router = APIRouter(prefix="/chat", tags=["chat"])

class ChatRequest(BaseModel):
    message: str

class ChatResponse(BaseModel):
    message: str
    timestamp: str

# Define tools with access to database
def create_tools(session: Session, user_id: int):
    """Factory function to create tools with user context."""

    @function_tool
    async def get_user_tasks() -> list[dict]:
        """Get all tasks for the current user."""
        from sqlmodel import select
        query = select(Task).where(Task.user_id == user_id)
        tasks = session.exec(query).all()
        return [
            {
                "id": t.id,
                "title": t.title,
                "priority": t.priority,
                "is_complete": t.is_complete
            }
            for t in tasks
        ]

    @function_tool
    async def create_new_task(
        title: str,
        description: str = "",
        priority: str = "medium"
    ) -> dict:
        """Create a new task."""
        task = Task(
            user_id=user_id,
            title=title,
            description=description,
            priority=priority
        )
        session.add(task)
        session.commit()
        session.refresh(task)
        return {"id": task.id, "title": task.title, "status": "created"}

    @function_tool
    async def mark_complete(task_id: int) -> dict:
        """Mark a task as complete."""
        task = session.get(Task, task_id)
        if not task or task.user_id != user_id:
            return {"error": "Task not found"}
        task.is_complete = True
        session.add(task)
        session.commit()
        return {"id": task.id, "is_complete": True}

    return [get_user_tasks, create_new_task, mark_complete]

@router.post("/")
async def chat(
    request: ChatRequest,
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session)
) -> ChatResponse:
    """Chat with todo assistant."""

    try:
        # Create agent with user-specific tools
        tools = create_tools(session, current_user.id)
        agent = Agent(
            name="Todo Assistant",
            instructions="""You are a helpful todo list assistant.
Help users manage their tasks by:
- Showing their current tasks
- Creating new tasks
- Marking tasks complete

Be concise and confirm actions.""",
            model="gpt-4o",
            tools=tools
        )

        # Get session for conversation history
        chat_session = SQLiteSession(f"user_{current_user.id}")

        # Run agent with retry logic
        from tenacity import retry, wait_exponential, stop_after_attempt

        @retry(
            wait=wait_exponential(multiplier=1, min=1, max=60),
            stop=stop_after_attempt(3),
            reraise=True
        )
        async def run_agent():
            result = await Runner.run(agent, request.message, session=chat_session)
            return result

        result = await run_agent()

        return ChatResponse(
            message=result.final_output,
            timestamp=datetime.utcnow().isoformat()
        )

    except RateLimitError:
        raise HTTPException(status_code=429, detail="API rate limited")
    except APIError as e:
        raise HTTPException(status_code=502, detail="AI service error")
```

### Observability & Tracing

**Built-in Tracing**:
```python
from openai_agents import Agent, Runner

# Enable tracing (stores in ./.agent_traces by default)
agent = Agent(
    name="Assistant",
    instructions="...",
    tools=[...],
    # Tracing enabled by default
)

# View traces for debugging
# Files stored in: ./.agent_traces/
# Includes: tool calls, LLM responses, errors

# Access trace data programmatically
result = await Runner.run(agent, message, session=session)
print(f"Tools called: {result.tool_calls}")
print(f"Token usage: {result.usage}")
```

**Logging Best Practices**:
```python
import logging

logger = logging.getLogger("todo_agent")

@router.post("/chat")
async def chat(request: ChatRequest, current_user: User = Depends(get_current_user)):
    logger.info(f"User {current_user.id} message: {request.message}")

    try:
        result = await handle_chat(request.message, current_user.id)
        logger.info(f"Agent response: {result}")
        return ChatResponse(message=result)
    except Exception as e:
        logger.error(f"Chat error: {e}", exc_info=True)
        raise
```

### Implementation Checklist for Todo Chatbot

- ✅ Install `openai-agents` v0.6.4+ with `openai` v2.x
- ✅ Define function tools for: get_tasks, create_task, update_task, mark_complete, delete_task
- ✅ Create Agent with instructions + tools + model selection (gpt-4o for speed)
- ✅ Set up SQLiteSession for automatic conversation history per user
- ✅ Add error handling (RateLimitError, APITimeoutError, APIError) with retry logic
- ✅ Create FastAPI endpoint `/api/chat` accepting message + returning response
- ✅ Validate user owns tasks (user_id check in all tool functions)
- ✅ Log all agent interactions for debugging
- ✅ Test conversation flow: multi-turn context preservation
- ✅ Add rate limit handling (429 → wait and retry)
- ✅ Deploy with environment variable: `OPENAI_API_KEY`

---

## Open Questions / Clarifications Needed

None - all technical decisions have been researched and documented.

---

## Next Steps

1. Generate data-model.md with database schema
2. Generate API contracts in /contracts folder
3. Generate quickstart.md with setup instructions
4. Fill complete plan.md with all design artifacts
5. Implement OpenAI Agents SDK integration in backend/app/routers/chat.py
