# Implementation Plan: Phase-II Full-Stack Web Todo Application

**Branch**: `002-phase2-web` | **Date**: 2026-01-01 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/002-phase2-web/spec.md`

## Summary

Build a full-stack multi-user web todo application with Next.js 16+ frontend and FastAPI backend. The application provides user authentication via JWT tokens stored in httpOnly cookies, complete CRUD operations for tasks, and strict user isolation ensuring users can only access their own tasks. Data is persisted to Neon Serverless PostgreSQL database using SQLModel ORM. The frontend uses Next.js App Router with TypeScript and Tailwind CSS, while the backend implements RESTful API endpoints with automatic OpenAPI documentation.

**Key Technical Approach**:
- JWT authentication with httpOnly cookies for XSS protection
- Row-level user isolation via user_id foreign key filtering
- SQLModel for unified database models and API schemas
- Next.js middleware for client-side route protection
- FastAPI dependency injection for current user context
- Connection pooling for Neon serverless database

## Technical Context

**Language/Version**:
- Backend: Python 3.11+
- Frontend: TypeScript 5.3+ (strict mode)

**Primary Dependencies**:
- Backend: FastAPI 0.104+, SQLModel 0.0.14+, python-jose[cryptography], passlib[bcrypt], uvicorn
- Frontend: Next.js 16+, React 19+, Tailwind CSS 3.4+

**Storage**: Neon Serverless PostgreSQL (connection string in environment variable)

**Testing**:
- Backend: pytest with FastAPI TestClient (for future testing)
- Frontend: Manual testing via browser, future: React Testing Library

**Target Platform**:
- Backend: Linux server (Railway/Render/Fly.io for deployment)
- Frontend: Vercel (Next.js optimized platform)
- Database: Neon Serverless PostgreSQL (already cloud-hosted)

**Project Type**: Web application (separate backend and frontend directories)

**Performance Goals**:
- API response time: < 200ms p95 latency
- Dashboard load time: < 1 second
- Support 100+ concurrent users
- Database queries optimized with indexes

**Constraints**:
- Phase-I CLI code (`src/`) must NOT be modified
- JWT tokens expire after 24 hours
- Passwords hashed with bcrypt (cost factor 12)
- User isolation enforced at database and API layer
- CORS configured for frontend origin only

**Scale/Scope**:
- Expected users: 100-1000
- Tasks per user: 10-1000
- Total database records: ~100K tasks, ~1K users

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### Phase-I Constitution Compliance (CLI Todo App)

The Phase-I constitution governs the CLI application in `src/` directory:
- ✅ **Spec-Driven Development**: Phase-I followed spec → plan → tasks workflow
- ✅ **Test-First Development**: Phase-I has acceptance scenarios and test cases
- ✅ **Clean Code**: Phase-I uses PEP 8, type hints, docstrings
- ✅ **Modular Architecture**: Phase-I separates models, services, and CLI
- ✅ **In-Memory Storage**: Phase-I uses in-memory data structures (no persistence)
- ✅ **Zero External Dependencies**: Phase-I uses only Python stdlib

**Gate Status**: ✅ PASS - Phase-I remains unchanged and compliant

### Phase-II Constitution Alignment

Phase-II introduces new architecture patterns while respecting Phase-I's existence:

**Architectural Principles**:
- ✅ **Separation of Concerns**: Backend and frontend are separate projects
- ✅ **Database Persistence**: Replaces in-memory storage (evolutionary step from Phase-I)
- ✅ **Multi-User Support**: Adds authentication and user isolation (new capability)
- ✅ **External Dependencies**: Justified for web framework, database, authentication (necessary for web app)
- ✅ **RESTful API Design**: Standard patterns, well-documented via OpenAPI
- ✅ **Security First**: Password hashing, JWT, input validation, user isolation

**Code Quality Standards**:
- ✅ **Type Safety**: TypeScript (frontend), Python type hints (backend)
- ✅ **Code Organization**: Modular structure with clear responsibilities
- ✅ **Documentation**: Comprehensive API docs, code comments, README
- ✅ **Error Handling**: Graceful error handling, user-friendly messages
- ✅ **Validation**: Frontend and backend validation layers

**Spec-Driven Development**:
- ✅ **Specification Complete**: spec.md defines all requirements
- ✅ **Design Artifacts**: research.md, data-model.md, contracts/, quickstart.md
- ✅ **Implementation Plan**: This file (plan.md) provides architecture
- ✅ **Task Breakdown**: tasks.md will follow (created by /sp.tasks command)

**Gate Status**: ✅ PASS - Phase-II follows spec-driven workflow and establishes appropriate architecture for web application

### Non-Compliance / Justifications

**None - No constitutional violations detected**

Phase-II introduces new technology stack (web framework, database) but this is appropriate evolution from CLI to web application. The Phase-I constitution was specific to the CLI app's constraints (in-memory storage, zero dependencies). Phase-II establishes its own architectural patterns suitable for multi-user web applications.

## Project Structure

### Documentation (this feature)

```text
specs/002-phase2-web/
├── spec.md              # Feature specification (COMPLETE)
├── plan.md              # This file - implementation plan (COMPLETE)
├── research.md          # Phase 0 output - technology decisions (COMPLETE)
├── data-model.md        # Phase 1 output - database schema (COMPLETE)
├── quickstart.md        # Phase 1 output - setup guide (COMPLETE)
├── contracts/           # Phase 1 output - API contracts (COMPLETE)
│   └── api-spec.md      # RESTful API specification
└── tasks.md             # Phase 2 output - NOT created by /sp.plan (PENDING /sp.tasks)
```

### Source Code (repository root)

```text
to-do/                              # Repository root
├── src/                            # Phase-I CLI code (DO NOT MODIFY)
│   ├── __init__.py
│   ├── models.py                   # CLI Task model
│   ├── todo_service.py             # CLI business logic
│   └── main.py                     # CLI entry point
│
├── backend/                        # Phase-II FastAPI backend (NEW)
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                 # FastAPI app entry point
│   │   ├── database.py             # Database connection and session
│   │   ├── models.py               # SQLModel models (User, Task)
│   │   ├── schemas.py              # Pydantic request/response schemas
│   │   ├── auth.py                 # Authentication logic (JWT, hashing)
│   │   ├── dependencies.py         # FastAPI dependencies (get_current_user)
│   │   └── routers/
│   │       ├── __init__.py
│   │       ├── auth.py             # Auth endpoints (register, login, logout)
│   │       └── tasks.py            # Task CRUD endpoints
│   ├── alembic/                    # Database migrations (optional for MVP)
│   │   ├── versions/
│   │   ├── env.py
│   │   └── alembic.ini
│   ├── tests/                      # Backend tests (future)
│   │   ├── __init__.py
│   │   ├── test_auth.py
│   │   └── test_tasks.py
│   ├── requirements.txt            # Python dependencies
│   ├── .env.example                # Environment variables template
│   └── README.md                   # Backend setup instructions
│
├── frontend/                       # Phase-II Next.js frontend (NEW)
│   ├── app/                        # Next.js App Router
│   │   ├── layout.tsx              # Root layout with Navbar
│   │   ├── page.tsx                # Landing page (redirect to dashboard)
│   │   ├── register/
│   │   │   └── page.tsx            # Registration page (client component)
│   │   ├── login/
│   │   │   └── page.tsx            # Login page (client component)
│   │   └── dashboard/
│   │       └── page.tsx            # Dashboard with task list (server component)
│   ├── components/                 # React components
│   │   ├── TaskList.tsx            # Client component for task list
│   │   ├── TaskItem.tsx            # Client component for task row
│   │   ├── TaskForm.tsx            # Client component for add/edit modal
│   │   └── Navbar.tsx              # Client component for navigation
│   ├── lib/                        # Utility functions
│   │   ├── api.ts                  # API client functions
│   │   ├── auth.ts                 # Auth helper functions
│   │   └── types.ts                # TypeScript type definitions
│   ├── middleware.ts               # Route protection middleware
│   ├── public/                     # Static assets
│   ├── package.json
│   ├── tsconfig.json
│   ├── tailwind.config.js
│   ├── next.config.js
│   ├── .env.example
│   └── README.md                   # Frontend setup instructions
│
├── specs/                          # Feature specifications (EXISTING)
│   ├── 001-cli-todo-app/           # Phase-I spec (existing)
│   └── 002-phase2-web/             # Phase-II spec (this feature)
│
├── .specify/                       # Spec-Kit Plus templates (EXISTING)
├── history/                        # PHR and ADR records (EXISTING)
├── README.md                       # Project root README (UPDATE for Phase-II)
├── CLAUDE.md                       # AI generation prompts (EXISTING)
└── .gitignore                      # Git ignore patterns
```

**Structure Decision**:

Web application structure (Option 2 from template) is chosen because:
1. **Separation of Concerns**: Backend and frontend are independent projects with different tech stacks
2. **Independent Deployment**: Backend and frontend can be deployed to different platforms (Railway/Vercel)
3. **Clear Boundaries**: API contract (REST) defines communication interface
4. **Scalability**: Each component can scale independently
5. **Development Workflow**: Frontend and backend teams can work in parallel

The `backend/` and `frontend/` directories are top-level to signify they are separate projects. Phase-I CLI code in `src/` remains untouched as per specification constraints.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No violations detected - this section is not needed.

---

## Architecture Overview

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                          Browser                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Next.js Frontend (localhost:3000)                       │   │
│  │  ┌────────────────────────────────────────────────────┐  │   │
│  │  │  Pages (App Router)                                │  │   │
│  │  │  - / (landing/redirect)                            │  │   │
│  │  │  - /register (UserRegisterForm)                    │  │   │
│  │  │  - /login (UserLoginForm)                          │  │   │
│  │  │  - /dashboard (TaskList, TaskForm)                 │  │   │
│  │  └────────────────────────────────────────────────────┘  │   │
│  │  ┌────────────────────────────────────────────────────┐  │   │
│  │  │  Middleware (middleware.ts)                        │  │   │
│  │  │  - Check JWT cookie                                │  │   │
│  │  │  - Redirect unauthenticated users                  │  │   │
│  │  │  - Redirect authenticated users from auth pages    │  │   │
│  │  └────────────────────────────────────────────────────┘  │   │
│  └──────────────────┬───────────────────────────────────────┘   │
│                     │                                            │
│                     │ HTTP Requests + JWT Cookie (httpOnly)      │
│                     │                                            │
└─────────────────────┼────────────────────────────────────────────┘
                      │
                      ▼
         ┌────────────────────────────────┐
         │  CORS Middleware               │
         │  - Verify origin               │
         │  - Allow credentials (cookies) │
         └────────────┬───────────────────┘
                      │
         ┌────────────▼─────────────────────────────────────┐
         │  FastAPI Backend (localhost:8000)                │
         │                                                   │
         │  ┌────────────────────────────────────────────┐  │
         │  │  /api/auth Router (routers/auth.py)       │  │
         │  │  - POST /register (create user, hash pwd) │  │
         │  │  - POST /login (verify pwd, issue JWT)    │  │
         │  │  - POST /logout (clear JWT cookie)        │  │
         │  └────────────────────────────────────────────┘  │
         │                                                   │
         │  ┌────────────────────────────────────────────┐  │
         │  │  JWT Middleware (dependencies.py)          │  │
         │  │  - Extract JWT from cookie                 │  │
         │  │  - Verify signature                        │  │
         │  │  - Decode payload → user_id                │  │
         │  │  - Load User from database                 │  │
         │  └──────────────┬─────────────────────────────┘  │
         │                 │ current_user injected           │
         │                 ▼                                 │
         │  ┌────────────────────────────────────────────┐  │
         │  │  /api/tasks Router (routers/tasks.py)      │  │
         │  │  - GET /tasks (filter by user_id)          │  │
         │  │  - POST /tasks (set user_id from JWT)      │  │
         │  │  - PUT /tasks/{id} (verify ownership)      │  │
         │  │  - PATCH /tasks/{id}/toggle (ownership)    │  │
         │  │  - DELETE /tasks/{id} (verify ownership)   │  │
         │  └──────────────┬─────────────────────────────┘  │
         │                 │                                 │
         │                 ▼                                 │
         │  ┌────────────────────────────────────────────┐  │
         │  │  SQLModel ORM (models.py)                  │  │
         │  │  - User model (id, email, hashed_password) │  │
         │  │  - Task model (id, user_id, title, ...)    │  │
         │  │  - Relationships (User 1:N Task)           │  │
         │  └──────────────┬─────────────────────────────┘  │
         │                 │                                 │
         │                 │ SQL Queries (parameterized)     │
         └─────────────────┼─────────────────────────────────┘
                           │
                           ▼
         ┌──────────────────────────────────────────────┐
         │  Neon Serverless PostgreSQL                  │
         │  ┌────────────┐         ┌──────────────────┐ │
         │  │ users      │         │ tasks            │ │
         │  ├────────────┤         ├──────────────────┤ │
         │  │ id (PK)    │◄────────│ id (PK)          │ │
         │  │ email      │    1:N  │ user_id (FK)     │ │
         │  │ hashed_pwd │         │ title            │ │
         │  │ created_at │         │ description      │ │
         │  └────────────┘         │ priority         │ │
         │                         │ is_complete      │ │
         │                         │ created_at       │ │
         │                         │ updated_at       │ │
         │                         └──────────────────┘ │
         │                                              │
         │  Indexes:                                    │
         │  - users.email (UNIQUE)                      │
         │  - tasks.user_id (FK index)                  │
         │  - tasks.(user_id, created_at) (composite)   │
         └──────────────────────────────────────────────┘
```

### Authentication Flow

```
┌──────────────┐                                    ┌──────────────┐
│   Browser    │                                    │   Backend    │
└──────┬───────┘                                    └──────┬───────┘
       │                                                   │
       │  1. POST /api/auth/register                      │
       │     {email, password}                            │
       ├──────────────────────────────────────────────────>│
       │                                                   │
       │                         2. Hash password (bcrypt) │
       │                         3. INSERT INTO users      │
       │                         4. Generate JWT           │
       │                                                   │
       │  5. 201 Created + Set-Cookie: access_token       │
       │<──────────────────────────────────────────────────┤
       │                                                   │
       │  6. Navigate to /dashboard                        │
       ├──────────────────────────────────────────────────>│
       │     Cookie: access_token=<jwt>                    │
       │                                                   │
       │                         7. Decode JWT → user_id   │
       │                         8. SELECT tasks WHERE...  │
       │                                                   │
       │  9. 200 OK + tasks array                          │
       │<──────────────────────────────────────────────────┤
       │                                                   │
```

### Task CRUD Flow (with User Isolation)

```
┌──────────────┐                                    ┌──────────────┐
│   Browser    │                                    │   Backend    │
│  User A (1)  │                                    │              │
└──────┬───────┘                                    └──────┬───────┘
       │                                                   │
       │  1. POST /api/tasks                              │
       │     Cookie: access_token=<jwt_user_1>            │
       │     {title: "Buy milk", priority: "high"}        │
       ├──────────────────────────────────────────────────>│
       │                                                   │
       │                         2. Decode JWT → user_id=1 │
       │                         3. INSERT INTO tasks      │
       │                            (user_id=1, ...)       │
       │                                                   │
       │  4. 201 Created + task object                     │
       │<──────────────────────────────────────────────────┤
       │                                                   │


┌──────────────┐                                    ┌──────────────┐
│   Browser    │                                    │   Backend    │
│  User B (2)  │                                    │              │
└──────┬───────┘                                    └──────┬───────┘
       │                                                   │
       │  1. GET /api/tasks/1                             │
       │     Cookie: access_token=<jwt_user_2>            │
       ├──────────────────────────────────────────────────>│
       │                                                   │
       │                         2. Decode JWT → user_id=2 │
       │                         3. SELECT * FROM tasks    │
       │                            WHERE id=1             │
       │                         4. Check task.user_id=1   │
       │                         5. user_id != current_user│
       │                                                   │
       │  6. 403 Forbidden                                 │
       │     {"detail": "Not authorized"}                  │
       │<──────────────────────────────────────────────────┤
       │                                                   │
```

---

## Implementation Phases

### Phase 0: Research & Technology Selection ✅ COMPLETE

**Status**: ✅ COMPLETE (research.md generated)

**Artifacts**:
- ✅ research.md with technology decisions and rationale

**Decisions Made**:
1. Next.js 16+ App Router for frontend architecture
2. FastAPI + SQLModel for backend with unified models
3. python-jose for JWT, passlib[bcrypt] for password hashing
4. Neon Serverless PostgreSQL with connection pooling
5. httpOnly cookies for JWT storage (XSS protection)
6. Row-level user isolation via user_id filtering

---

### Phase 1: Data Model & API Contracts ✅ COMPLETE

**Status**: ✅ COMPLETE (data-model.md, contracts/, quickstart.md generated)

**Artifacts**:
- ✅ data-model.md with database schema, relationships, validation
- ✅ contracts/api-spec.md with RESTful API specification
- ✅ quickstart.md with setup instructions

**Database Schema Defined**:
- Users table: id, email, hashed_password, created_at
- Tasks table: id, user_id (FK), title, description, priority, is_complete, created_at, updated_at
- Relationships: User 1:N Task (CASCADE delete)
- Indexes: users.email (unique), tasks.user_id, composite (user_id, created_at)

**API Endpoints Specified**:
- Auth: POST /register, POST /login, POST /logout
- Tasks: GET /tasks, POST /tasks, GET /tasks/{id}, PUT /tasks/{id}, PATCH /tasks/{id}/toggle, DELETE /tasks/{id}

---

### Phase 2: Task Breakdown 🔜 NEXT

**Status**: 🔜 PENDING (requires /sp.tasks command)

**Output**: tasks.md with atomic, ordered implementation tasks

**Not created by /sp.plan - this is generated by /sp.tasks command**

---

## Key Design Decisions

### 1. Authentication Strategy: JWT in httpOnly Cookies

**Decision**: Use JWT tokens stored in httpOnly cookies instead of localStorage or Authorization header.

**Rationale**:
- **XSS Protection**: httpOnly flag prevents JavaScript access to token
- **CSRF Protection**: SameSite=Lax cookie attribute mitigates CSRF attacks
- **Automatic Sending**: Browser automatically includes cookies in requests
- **Standard Practice**: Industry standard for web applications

**Alternatives Rejected**:
- localStorage: Vulnerable to XSS attacks
- Authorization header: Requires manual token management, less secure

**Implementation**:
```python
# Backend: Set cookie on login
response.set_cookie(
    key="access_token",
    value=jwt_token,
    httponly=True,
    secure=True,  # HTTPS only
    samesite="lax",
    max_age=86400  # 24 hours
)
```

```typescript
// Frontend: Cookies sent automatically
fetch('/api/tasks', {
  credentials: 'include'  // Include cookies
})
```

---

### 2. User Isolation: Row-Level Filtering via user_id

**Decision**: Enforce user isolation at database query level by filtering all queries with `WHERE user_id = current_user.id`.

**Rationale**:
- **Security**: Prevents users from accessing other users' data
- **Database Constraint**: Foreign key ensures referential integrity
- **Simple Implementation**: Single WHERE clause in all queries
- **Performance**: Index on user_id makes filtering efficient

**Alternatives Rejected**:
- Application-level filtering only: Vulnerable to bugs
- Database row-level security policies: More complex, overkill for MVP
- Separate tables per user: Doesn't scale, poor design

**Implementation**:
```python
# Every task query includes user_id filter
tasks = session.exec(
    select(Task).where(Task.user_id == current_user.id)
).all()

# Ownership verification on updates/deletes
task = session.get(Task, task_id)
if task.user_id != current_user.id:
    raise HTTPException(403, "Not authorized")
```

---

### 3. ORM Choice: SQLModel (SQLAlchemy + Pydantic)

**Decision**: Use SQLModel for database models instead of separate SQLAlchemy models + Pydantic schemas.

**Rationale**:
- **Less Boilerplate**: Single model definition for both database and API
- **Type Safety**: Full type checking with Python type hints
- **Validation**: Pydantic validation built-in
- **FastAPI Integration**: Designed specifically for FastAPI
- **Maintainability**: Fewer files to keep in sync

**Alternatives Rejected**:
- SQLAlchemy + Pydantic separately: Double definitions, more boilerplate
- Django ORM: Requires full Django framework
- Prisma: Less mature in Python, different migration approach

**Example**:
```python
from sqlmodel import Field, SQLModel

class Task(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    user_id: int = Field(foreign_key="users.id")
    title: str = Field(min_length=1, max_length=200)
    # ... serves both as database model AND API schema
```

---

### 4. Frontend Architecture: Next.js App Router with Server Components

**Decision**: Use Next.js 16+ App Router with mix of Server and Client Components.

**Rationale**:
- **Performance**: Server Components reduce client-side JavaScript
- **SEO**: Server-side rendering improves search indexing
- **Developer Experience**: Built-in routing, layouts, loading states
- **Type Safety**: Excellent TypeScript support
- **React 19 Features**: Concurrent rendering, automatic batching

**Server Components** (default):
- Dashboard page (initial data fetching)
- Layouts
- Static pages

**Client Components** (`'use client'`):
- Forms (registration, login, task creation)
- Interactive task list (checkboxes, delete buttons)
- Modals

**Alternatives Rejected**:
- Pages Router: Older pattern, less performant
- Pure SPA (Vite + React): No SSR, worse SEO
- Remix: Less mature ecosystem

---

### 5. Password Security: bcrypt with Cost Factor 12

**Decision**: Hash passwords with bcrypt algorithm at cost factor 12.

**Rationale**:
- **Industry Standard**: bcrypt is battle-tested for password hashing
- **Adaptive**: Cost factor can be increased as hardware improves
- **Salted**: Automatic per-password salt prevents rainbow tables
- **Secure**: Resistant to brute-force attacks

**Alternatives Rejected**:
- argon2: Good choice but bcrypt more widely used, simpler setup
- SHA-256: NOT suitable for passwords (too fast, no salt)
- MD5: Completely broken, never use

**Implementation**:
```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# On registration
hashed = pwd_context.hash(plain_password)  # Cost factor 12 default

# On login
is_valid = pwd_context.verify(plain_password, hashed)
```

---

### 6. API Design: RESTful with Standard HTTP Methods

**Decision**: Use RESTful API design with standard HTTP methods and status codes.

**Rationale**:
- **Industry Standard**: Well-understood by all developers
- **Predictable**: Standard patterns for CRUD operations
- **Tooling**: Works with all HTTP clients, debugging tools
- **Documentation**: OpenAPI/Swagger auto-generation

**REST Conventions**:
- GET /tasks → List tasks (200 OK)
- POST /tasks → Create task (201 Created)
- PUT /tasks/{id} → Update task (200 OK)
- PATCH /tasks/{id}/toggle → Partial update (200 OK)
- DELETE /tasks/{id} → Delete task (204 No Content or 200 OK)

**Status Codes**:
- 2xx: Success (200 OK, 201 Created, 204 No Content)
- 4xx: Client error (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found)
- 5xx: Server error (500 Internal Server Error, 503 Service Unavailable)

**Alternatives Rejected**:
- GraphQL: Overkill for simple CRUD, adds complexity
- RPC-style endpoints: Non-standard, harder to document
- Custom protocols: Reinventing the wheel

---

## Risk Analysis

### Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Neon database cold start latency | Medium | Low | Connection pooling with `pool_pre_ping=True`, accept slight delay for serverless benefits |
| JWT token theft via XSS | High | Low | httpOnly cookies, React auto-escaping, input validation |
| CORS misconfiguration allowing unauthorized origins | High | Low | Explicit CORS_ORIGINS whitelist, test with different origins |
| Password hash cost too low (brute force) | High | Low | bcrypt cost factor 12 (industry standard), monitor hash time |
| SQL injection via user input | High | Low | SQLModel parameterized queries, never use string interpolation |
| User enumeration via login errors | Medium | Low | Same error message for wrong email vs wrong password |
| Session fixation attacks | Medium | Low | Generate new JWT on login, short expiration (24 hours) |

### Implementation Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Forgetting user_id filter in queries | High | Medium | Code review, create query helper functions, add tests |
| Frontend/backend type mismatches | Medium | Medium | Share TypeScript types via OpenAPI code generation |
| CORS issues in deployment | Medium | Medium | Test with production URLs before deployment, document CORS setup |
| Environment variables not set | High | Low | .env.example files, validation on startup, clear error messages |
| Database migration failures | Medium | Low | Use Alembic for migrations, test on staging first |

### Security Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| JWT secret leaked | Critical | Low | Store in environment variable, never commit to git, rotate periodically |
| Database credentials leaked | Critical | Low | Store in environment variable, use .gitignore, Neon IAM if available |
| User tries to access another user's task | High | Medium | Ownership check on all task operations, return 403 Forbidden |
| Brute force login attempts | Medium | Medium | Rate limiting (future), monitor failed login attempts |
| XSS via task title/description | Medium | Low | React auto-escapes, validate max lengths, sanitize on display |

---

## Testing Strategy

### Backend Testing (Future)

**Unit Tests** (pytest):
- `test_auth.py`: Registration, login, password hashing
- `test_tasks.py`: CRUD operations, user isolation
- `test_models.py`: Database constraints, validation

**Integration Tests**:
- End-to-end API workflows
- Database transaction rollback after each test

**Test User Isolation**:
```python
def test_user_cannot_access_other_user_task():
    # Create User A and Task 1
    # Login as User B
    # Attempt to GET /tasks/1
    # Assert 403 Forbidden
```

### Frontend Testing (Future)

**Component Tests** (React Testing Library):
- Form validation (registration, login, task creation)
- Task list rendering
- Button interactions

**E2E Tests** (Playwright):
- Complete user journey (register → create task → logout → login)
- User isolation verification

### Manual Testing Checklist

- [ ] Register new user with valid email/password
- [ ] Register with duplicate email (should fail)
- [ ] Login with correct credentials
- [ ] Login with wrong password (should fail)
- [ ] Create task while authenticated
- [ ] Create task without authentication (should redirect)
- [ ] View task list (should show only my tasks)
- [ ] Update task title/description/priority
- [ ] Toggle task completion status
- [ ] Delete task
- [ ] Attempt to access another user's task (should fail)
- [ ] Logout and verify session cleared

---

## Deployment Considerations (Future)

### Backend Deployment

**Platform**: Railway, Render, or Fly.io

**Environment Variables**:
```env
DATABASE_URL=postgresql://...@neon.tech/...
JWT_SECRET_KEY=<generate-new-secret-for-production>
CORS_ORIGINS=https://your-frontend.vercel.app
ENVIRONMENT=production
```

**Startup Command**:
```bash
uvicorn app.main:app --host 0.0.0.0 --port $PORT
```

**Health Check Endpoint**:
```python
@app.get("/health")
def health():
    return {"status": "ok"}
```

### Frontend Deployment

**Platform**: Vercel (Next.js optimized)

**Environment Variables**:
```env
NEXT_PUBLIC_API_URL=https://your-backend.railway.app/api
```

**Build Command**:
```bash
npm run build
```

**Start Command**:
```bash
npm start
```

### Database

Neon is already serverless - use production connection string.

**Backup Strategy**: Neon provides automatic backups

---

## Success Criteria

Implementation is considered successful when:

### Functional Requirements ✅
- [ ] User can register with email and password
- [ ] User can login and receive JWT token in cookie
- [ ] User can view dashboard with their tasks only
- [ ] User can create new tasks with title, description, priority
- [ ] User can update existing tasks
- [ ] User can toggle task completion status
- [ ] User can delete tasks
- [ ] User can logout and session is cleared
- [ ] User isolation is 100% enforced (users never see other users' tasks)

### Security Requirements ✅
- [ ] Passwords are hashed with bcrypt before storage
- [ ] JWT tokens are signed and validated
- [ ] Tokens are stored in httpOnly cookies
- [ ] CORS is configured to allow only frontend origin
- [ ] Input validation on both frontend and backend
- [ ] SQL injection prevented via parameterized queries
- [ ] XSS prevented via React escaping and validation

### Performance Requirements ✅
- [ ] API endpoints respond in < 200ms p95
- [ ] Dashboard loads in < 1 second
- [ ] Task operations (create/update/delete) complete in < 500ms

### Code Quality Requirements ✅
- [ ] Backend follows PEP 8 style guide
- [ ] Backend uses type hints for all functions
- [ ] Frontend uses TypeScript strict mode
- [ ] API has OpenAPI documentation at /docs
- [ ] Environment variables are documented in .env.example
- [ ] Setup instructions in quickstart.md work on fresh system

---

## Next Steps

1. **Review Planning Artifacts**:
   - ✅ spec.md (feature specification)
   - ✅ plan.md (this file)
   - ✅ research.md (technology decisions)
   - ✅ data-model.md (database schema)
   - ✅ contracts/api-spec.md (API specification)
   - ✅ quickstart.md (setup guide)

2. **Generate Task Breakdown**:
   - Run `/sp.tasks` command to generate tasks.md
   - Tasks will be atomic, ordered, with acceptance criteria

3. **Execute Implementation**:
   - Run `/sp.implement` command to execute tasks
   - Follow TDD workflow (red-green-refactor)

4. **Commit and PR**:
   - Run `/sp.git.commit_pr` to commit changes and create pull request

---

## Conclusion

This implementation plan provides a comprehensive architecture for Phase-II web todo application. All technology decisions have been researched and documented. The database schema, API contracts, and setup instructions are complete. The next step is to generate the task breakdown (tasks.md) using the `/sp.tasks` command, followed by implementation via `/sp.implement`.

**Key Highlights**:
- Multi-user authentication with JWT
- Strict user isolation via database filtering
- Modern tech stack (Next.js 16+, FastAPI, Neon PostgreSQL)
- Security-first approach (password hashing, httpOnly cookies, input validation)
- RESTful API with automatic documentation
- Clear separation between backend and frontend
- Phase-I CLI code remains untouched

**Estimated Complexity**: Medium
- Backend: ~1200 lines of Python
- Frontend: ~800 lines of TypeScript/React
- Database: 2 tables with relationships
- Total: ~2000 lines of code

**Implementation Time Estimate**: N/A (per constitution, no timeline estimates provided)

The plan is complete and ready for task breakdown phase.
