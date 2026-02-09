# Feature Specification: Phase-II Full-Stack Web Todo Application

**Feature Branch**: `002-phase2-web`
**Created**: 2026-01-01
**Status**: Draft
**Input**: User requirement for full-stack web-based multi-user todo application

## User Scenarios & Testing *(mandatory)*

### User Story 1 - User Registration (Priority: P1) 🎯 MVP

As a new user, I want to register for an account with email and password so that I can access my personal todo list.

**Why this priority**: Foundation for multi-user system. Without user accounts, we cannot implement user isolation or authentication. This is the entry point for all users.

**Independent Test**: Can be fully tested by navigating to /register, submitting email "test@example.com" and password "SecurePass123!", and verifying account creation with success message. User should then exist in database.

**Acceptance Scenarios**:

1. **Given** I am on the registration page, **When** I enter valid email "user@example.com" and password "SecurePass123!", **Then** the system creates my account, generates a JWT token, and redirects me to the dashboard

2. **Given** I am registering, **When** I enter an email that already exists in the system, **Then** the system displays "Error: Email already registered" and does not create a duplicate account

3. **Given** I am on the registration page, **When** I enter an invalid email format like "notanemail", **Then** the system displays "Error: Invalid email format" before form submission

4. **Given** I am registering, **When** I enter a password shorter than 8 characters, **Then** the system displays "Error: Password must be at least 8 characters" and prevents registration

5. **Given** I successfully register, **When** the system redirects me to the dashboard, **Then** I am automatically authenticated with a valid JWT token stored in httpOnly cookie

---

### User Story 2 - User Login (Priority: P1) 🎯 MVP

As a registered user, I want to log in with my email and password so that I can access my personal todo list.

**Why this priority**: Essential for returning users. Works with US1 to form complete authentication flow. Without login, users cannot access their data after initial registration.

**Independent Test**: Can be tested by first registering a user (US1), logging out, then logging in with same credentials and verifying successful authentication and JWT token issuance.

**Acceptance Scenarios**:

1. **Given** I have a registered account with email "user@example.com", **When** I enter correct email and password on login page, **Then** the system authenticates me, issues a JWT token, and redirects to dashboard

2. **Given** I am on the login page, **When** I enter a valid email but incorrect password, **Then** the system displays "Error: Invalid credentials" without revealing whether email exists

3. **Given** I am logging in, **When** I enter an email that doesn't exist in the system, **Then** the system displays "Error: Invalid credentials" (same message as wrong password for security)

4. **Given** I successfully log in, **When** I navigate to protected routes, **Then** my JWT token is included in requests and validated by the backend

5. **Given** I am already logged in, **When** I try to access the login page, **Then** the system redirects me to the dashboard automatically

---

### User Story 3 - View Personal Dashboard (Priority: P1) 🎯 MVP

As a logged-in user, I want to see my personal dashboard with my todo list so that I can view all my tasks in one place.

**Why this priority**: Primary interface after authentication. Must display user-isolated tasks. Without this, users cannot interact with their data. Forms core MVP with US1, US2, and US4.

**Independent Test**: Can be tested by logging in, verifying dashboard displays only tasks belonging to authenticated user, and confirming other users' tasks are not visible.

**Acceptance Scenarios**:

1. **Given** I am logged in with 3 tasks (2 incomplete, 1 complete), **When** I view my dashboard, **Then** the system displays all 3 of MY tasks in a table/card layout with ID, priority, status, title, and description

2. **Given** I am logged in but have no tasks, **When** I view my dashboard, **Then** the system displays "No tasks yet. Create your first task!" with a prominent "Add Task" button

3. **Given** I am logged in and another user has 10 tasks, **When** I view my dashboard, **Then** I see ONLY my tasks, never tasks belonging to other users (strict user isolation)

4. **Given** I am not authenticated (no valid JWT), **When** I try to access the dashboard, **Then** the system redirects me to the login page with message "Please log in to continue"

5. **Given** I am viewing my dashboard, **When** tasks are displayed, **Then** they are sorted by creation date (newest first) with visual indicators for priority (High=red, Medium=yellow, Low=green)

---

### User Story 4 - Create New Task (Priority: P1) 🎯 MVP

As a logged-in user, I want to create a new task with title, description, and priority so that I can track things I need to do.

**Why this priority**: Core functionality. Without task creation, the application has no value. Works with US3 to form minimal viable product for task management.

**Independent Test**: Can be tested by logging in, clicking "Add Task", filling form with title "Buy groceries", description "Milk, eggs, bread", priority "high", and verifying task appears in dashboard with correct user association.

**Acceptance Scenarios**:

1. **Given** I am on the dashboard, **When** I click "Add Task" button and fill form with title "Buy milk", description "Whole milk 2L", priority "high", **Then** the system creates task associated with my user ID, assigns unique ID, sets status to incomplete, and displays success message

2. **Given** I am creating a task, **When** I submit with empty title, **Then** the system displays "Error: Title is required" and prevents submission

3. **Given** I am creating a task, **When** I submit with only title "Call dentist" (no description, default priority), **Then** the system creates task with empty description and priority "medium"

4. **Given** I create a task, **When** the task is saved, **Then** it appears immediately in my dashboard without requiring page refresh (optimistic UI update)

5. **Given** I am creating a task, **When** the API request fails (network error), **Then** the system displays "Error: Failed to create task. Please try again" and does not show the task in UI

---

### User Story 5 - Update Task Details (Priority: P2)

As a logged-in user, I want to edit a task's title, description, or priority so that I can update information as it changes.

**Why this priority**: Important for task maintenance but not essential for MVP. Users can work around by deleting and recreating. Should come after core CRUD operations.

**Independent Test**: Can be tested by creating a task (US4), clicking "Edit" button, changing title from "Buy milk" to "Buy almond milk", and verifying change persists and is immediately visible in dashboard.

**Acceptance Scenarios**:

1. **Given** I have a task with title "Buy milk", **When** I click edit, change title to "Buy almond milk", and save, **Then** the system updates the task and displays "Task updated successfully"

2. **Given** I am editing a task, **When** I change priority from "medium" to "high", **Then** the system updates priority and task visual indicator changes to red

3. **Given** I try to edit a task, **When** I clear the title field and submit, **Then** the system displays "Error: Title cannot be empty" and preserves original data

4. **Given** I am editing a task, **When** another user tries to edit my task via API (direct request), **Then** the system returns 403 Forbidden error (user isolation enforcement)

5. **Given** I edit a task, **When** I close the edit form without saving, **Then** the system discards changes and shows original task data

---

### User Story 6 - Toggle Task Completion (Priority: P2)

As a logged-in user, I want to mark tasks as complete or incomplete with a single click so that I can track my progress.

**Why this priority**: Primary workflow action for task management. More frequently used than edit/delete. Essential for tracking completion status.

**Independent Test**: Can be tested by creating an incomplete task, clicking the checkbox/toggle button, verifying task shows as complete with visual indicator (strikethrough, checkmark), then toggling back to incomplete.

**Acceptance Scenarios**:

1. **Given** I have an incomplete task, **When** I click the checkbox/toggle button, **Then** the system marks it complete, updates status in database, and shows visual feedback (checkmark, strikethrough)

2. **Given** I have a complete task, **When** I click the checkbox/toggle button, **Then** the system marks it incomplete and removes completion visual indicators

3. **Given** I toggle a task, **When** the status changes, **Then** the change persists across page refreshes and browser sessions

4. **Given** I toggle a task, **When** the API request is in progress, **Then** the UI shows loading state and prevents duplicate toggles

5. **Given** I toggle a task, **When** another user tries to toggle my task via API, **Then** the system returns 403 Forbidden (user isolation)

---

### User Story 7 - Delete Task (Priority: P3)

As a logged-in user, I want to delete tasks that are no longer relevant so that my task list stays clean.

**Why this priority**: Useful for cleanup but not essential for core workflow. Lowest priority as users can simply ignore irrelevant tasks. Should be implemented last.

**Independent Test**: Can be tested by creating a task, clicking delete button, confirming deletion in modal, and verifying task is removed from dashboard and database.

**Acceptance Scenarios**:

1. **Given** I have a task, **When** I click delete and confirm in confirmation modal, **Then** the system permanently deletes the task and displays "Task deleted successfully"

2. **Given** I click delete, **When** the confirmation modal appears, I click "Cancel", **Then** the system does not delete the task and closes the modal

3. **Given** I delete a task, **When** the task is removed, **Then** it disappears from the dashboard immediately without page refresh

4. **Given** I try to delete a task, **When** another user tries to delete my task via API, **Then** the system returns 403 Forbidden error

5. **Given** I delete a task, **When** I try to access that task by ID later, **Then** the system returns 404 Not Found error

---

### User Story 8 - User Logout (Priority: P2)

As a logged-in user, I want to log out of my account so that others cannot access my tasks on a shared device.

**Why this priority**: Important for security, especially on shared computers. Should be implemented alongside authentication flow.

**Independent Test**: Can be tested by logging in, clicking logout button, and verifying JWT token is cleared, user is redirected to login page, and protected routes are no longer accessible.

**Acceptance Scenarios**:

1. **Given** I am logged in, **When** I click "Logout" button, **Then** the system clears my JWT token, redirects to login page, and displays "Successfully logged out"

2. **Given** I log out, **When** I try to access the dashboard, **Then** the system redirects me to login page with "Please log in to continue" message

3. **Given** I log out, **When** I use the browser back button, **Then** the system does not show my cached dashboard data and requires re-authentication

4. **Given** I am on any page, **When** I click logout, **Then** the logout action completes successfully regardless of current route

---

### User Story 9 - Filter and Sort Tasks (Priority: P4)

As a logged-in user, I want to filter tasks by status (all/complete/incomplete) and sort by priority or date so that I can focus on relevant tasks.

**Why this priority**: Enhancement feature for better UX. Not essential for MVP. Can be added after core CRUD operations are stable.

**Independent Test**: Can be tested by creating multiple tasks with different statuses and priorities, then using filter dropdown to show only incomplete tasks and verifying complete tasks are hidden.

**Acceptance Scenarios**:

1. **Given** I have 5 tasks (3 incomplete, 2 complete), **When** I select filter "Incomplete only", **Then** the system displays only 3 incomplete tasks

2. **Given** I have tasks with mixed priorities, **When** I select sort by "Priority (High to Low)", **Then** the system displays high priority tasks first, then medium, then low

3. **Given** I apply filters, **When** I refresh the page, **Then** the system remembers my filter preferences (stored in local storage or URL params)

---

### Edge Cases

- **Expired JWT Token**: What happens when user's JWT expires while using the app? → Backend returns 401 Unauthorized, frontend intercepts and redirects to login with "Session expired, please log in again" message
- **Concurrent Task Updates**: What if user has two browser tabs open and updates same task in both? → Last write wins; consider adding optimistic locking with version field in future
- **Invalid User ID in JWT**: What if JWT is tampered with and contains non-existent user ID? → Backend validates user exists during token verification; returns 401 if invalid
- **Database Connection Failure**: How does system handle Neon database being temporarily unavailable? → FastAPI returns 503 Service Unavailable with error message; frontend displays "Service temporarily unavailable, please try again"
- **Very Long Task Titles/Descriptions**: What if user enters 10,000 character description? → Backend validates max length (title: 200 chars, description: 2000 chars); frontend enforces same limits with character counter
- **XSS Attack Attempts**: What if user enters `<script>alert('xss')</script>` in task title? → React automatically escapes HTML; backend also sanitizes input; content rendered as plain text
- **SQL Injection Attempts**: What if user enters `'; DROP TABLE tasks; --` in task title? → SQLModel uses parameterized queries; immune to SQL injection
- **Simultaneous User Registration**: What if two users try to register same email simultaneously? → Database unique constraint on email prevents duplicates; second request gets "Email already registered" error
- **Task Ownership Bypass**: What if malicious user tries to access another user's task by guessing task ID? → Backend validates task.user_id matches authenticated user ID; returns 403 Forbidden if mismatch

## Requirements *(mandatory)*

### Functional Requirements

#### Authentication & Authorization

- **FR-001**: System MUST allow users to register with email (unique) and password (minimum 8 characters)
- **FR-002**: System MUST hash passwords using bcrypt or argon2 before storing in database (never store plaintext)
- **FR-003**: System MUST authenticate users via email/password and issue JWT token on successful login
- **FR-004**: System MUST include user ID in JWT payload and sign tokens with secret key
- **FR-005**: System MUST validate JWT token on every API request to protected endpoints
- **FR-006**: System MUST implement httpOnly cookie for JWT storage (XSS protection)
- **FR-007**: System MUST set JWT expiration time (e.g., 24 hours) and enforce expiration on backend
- **FR-008**: System MUST allow users to log out by clearing JWT token on client side
- **FR-009**: System MUST redirect unauthenticated users to login page when accessing protected routes
- **FR-010**: System MUST prevent authenticated users from accessing login/register pages (auto-redirect to dashboard)

#### Task Management (Multi-User Isolated)

- **FR-011**: System MUST create tasks with required title, optional description, optional priority (high/medium/low, default: medium)
- **FR-012**: System MUST auto-assign unique sequential integer IDs to tasks per user (scoped to user, not global)
- **FR-013**: System MUST associate each task with user_id (foreign key to users table) on creation
- **FR-014**: System MUST set new task status to incomplete (is_complete = false) by default
- **FR-015**: System MUST allow users to view ONLY their own tasks (strict user isolation via WHERE user_id = current_user.id)
- **FR-016**: System MUST allow users to update title, description, and priority of their own tasks
- **FR-017**: System MUST allow users to toggle task completion status (incomplete ↔ complete)
- **FR-018**: System MUST allow users to delete their own tasks permanently
- **FR-019**: System MUST return 403 Forbidden error if user tries to access/modify another user's task
- **FR-020**: System MUST persist all tasks to Neon PostgreSQL database (no in-memory storage)
- **FR-021**: System MUST validate task title is non-empty before creation/update
- **FR-022**: System MUST validate priority is one of: high, medium, low (case-insensitive)

#### API & Data Layer

- **FR-023**: Backend MUST expose RESTful API endpoints for all operations (register, login, CRUD tasks)
- **FR-024**: API MUST use FastAPI framework with automatic OpenAPI documentation at /docs
- **FR-025**: API MUST use SQLModel for ORM and database schema definition
- **FR-026**: API MUST return appropriate HTTP status codes (200, 201, 400, 401, 403, 404, 500)
- **FR-027**: API MUST return structured JSON responses with consistent error format: `{"detail": "error message"}`
- **FR-028**: Database MUST have users table with: id (PK), email (unique), hashed_password, created_at
- **FR-029**: Database MUST have tasks table with: id (PK), user_id (FK), title, description, priority, is_complete, created_at, updated_at
- **FR-030**: Database MUST enforce foreign key constraint: tasks.user_id → users.id with CASCADE delete
- **FR-031**: API MUST validate all input data using Pydantic models before processing
- **FR-032**: API MUST implement CORS middleware to allow requests from Next.js frontend

#### Frontend (Next.js)

- **FR-033**: Frontend MUST use Next.js 16+ with App Router (app/ directory structure)
- **FR-034**: Frontend MUST implement client-side routing with protected routes using middleware
- **FR-035**: Frontend MUST display registration form with email and password fields with validation
- **FR-036**: Frontend MUST display login form with email and password fields
- **FR-037**: Frontend MUST display dashboard with task list in table or card layout
- **FR-038**: Frontend MUST display "Add Task" button/form with fields for title, description, priority
- **FR-039**: Frontend MUST display edit button/modal for each task with pre-filled form
- **FR-040**: Frontend MUST display checkbox/toggle for task completion status
- **FR-041**: Frontend MUST display delete button with confirmation modal for each task
- **FR-042**: Frontend MUST show loading states during API requests
- **FR-043**: Frontend MUST show error messages for failed operations (network errors, validation errors)
- **FR-044**: Frontend MUST show success messages for successful operations (task created, updated, deleted)
- **FR-045**: Frontend MUST implement optimistic UI updates (show changes immediately before API confirmation)
- **FR-046**: Frontend MUST revert optimistic updates if API request fails

### Key Entities

- **User**: Represents a registered user account
  - Attributes: id (int, primary key), email (str, unique, indexed), hashed_password (str), created_at (datetime)
  - Relationships: One-to-many with Task (user can have multiple tasks)

- **Task**: Represents a single todo item belonging to a user
  - Attributes: id (int, primary key), user_id (int, foreign key), title (str, required), description (str, optional), priority (str, enum: high/medium/low), is_complete (bool, default: false), created_at (datetime), updated_at (datetime)
  - Relationships: Many-to-one with User (task belongs to one user)

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: User can complete full workflow (register → login → create task → view task → update task → toggle complete → delete task → logout) without errors
- **SC-002**: User isolation is 100% enforced - users can NEVER see or modify other users' tasks (verified by creating 2 users and attempting cross-access)
- **SC-003**: All API endpoints require valid JWT authentication (verified by testing without token and receiving 401 errors)
- **SC-004**: All passwords are hashed in database (verified by inspecting database - no plaintext passwords)
- **SC-005**: JWT tokens expire after configured time and system properly handles expired tokens (verified by waiting for expiration)
- **SC-006**: Frontend implements proper loading states and error handling for all async operations
- **SC-007**: Application prevents common security vulnerabilities (XSS, SQL injection, CSRF) - verified through basic security testing
- **SC-008**: API documentation is auto-generated and accessible at /docs endpoint with all endpoints documented
- **SC-009**: Database schema properly enforces constraints (unique email, foreign keys, non-null required fields)
- **SC-010**: Next.js frontend uses App Router with proper server/client component separation
- **SC-011**: All form inputs have proper validation on both frontend and backend
- **SC-012**: Application handles network errors gracefully without crashing or corrupting data

## Non-Functional Requirements

### Performance

- **NFR-001**: API endpoints respond in < 200ms for p95 latency under typical load (< 100 concurrent users)
- **NFR-002**: Dashboard loads initial task list in < 1 second
- **NFR-003**: Task creation/update/delete operations complete in < 500ms end-to-end
- **NFR-004**: Database connection pooling is configured for efficient connection reuse

### Security

- **NFR-005**: All passwords MUST be hashed using bcrypt with minimum cost factor of 12
- **NFR-006**: JWT tokens MUST be signed with strong secret key (minimum 256 bits, stored in environment variable)
- **NFR-007**: API MUST implement rate limiting to prevent brute force attacks (e.g., 5 failed login attempts per minute per IP)
- **NFR-008**: All API responses MUST include security headers (CORS, CSP, X-Content-Type-Options)
- **NFR-009**: Frontend MUST store JWT in httpOnly cookies (not localStorage) to prevent XSS theft
- **NFR-010**: Database connection string MUST be stored in environment variable, never committed to git
- **NFR-011**: API MUST validate and sanitize all user inputs to prevent injection attacks
- **NFR-012**: HTTPS MUST be enforced in production (HTTP redirected to HTTPS)

### Reliability

- **NFR-013**: Database transactions MUST be used for multi-step operations to ensure data consistency
- **NFR-014**: API MUST implement proper error handling and return meaningful error messages
- **NFR-015**: Frontend MUST implement retry logic for failed API requests (e.g., 3 retries with exponential backoff)
- **NFR-016**: System MUST gracefully handle database connection failures with appropriate error messages

### Usability

- **NFR-017**: All forms MUST provide clear, actionable error messages for validation failures
- **NFR-018**: Loading states MUST be shown for all async operations to provide user feedback
- **NFR-019**: Success messages MUST be displayed for all successful operations (auto-dismiss after 3 seconds)
- **NFR-020**: UI MUST be responsive and work on desktop, tablet, and mobile devices
- **NFR-021**: High priority tasks MUST have visual distinction (e.g., red badge, different color)

### Code Quality

- **NFR-022**: Backend code MUST follow PEP 8 style guidelines
- **NFR-023**: Backend code MUST use type hints for all function signatures
- **NFR-024**: Frontend code MUST use TypeScript with strict mode enabled
- **NFR-025**: All API endpoints MUST have Pydantic models for request/response validation
- **NFR-026**: Code MUST be organized following Clean Architecture principles (separation of concerns)
- **NFR-027**: Environment variables MUST be documented in .env.example file
- **NFR-028**: Database migrations MUST be versioned and tracked (using Alembic)

### Maintainability

- **NFR-029**: API endpoints MUST follow RESTful conventions (GET, POST, PUT, DELETE with appropriate routes)
- **NFR-030**: Frontend components MUST be modular and reusable
- **NFR-031**: Database schema MUST use migrations for version control (no manual schema changes)
- **NFR-032**: All configuration (database URL, JWT secret, CORS origins) MUST be environment-based

### Constraints

- **NFR-033**: Backend MUST use FastAPI framework (no Flask, Django, etc.)
- **NFR-034**: Frontend MUST use Next.js 16+ with App Router (no Pages Router)
- **NFR-035**: ORM MUST be SQLModel (no SQLAlchemy directly, no Prisma, etc.)
- **NFR-036**: Database MUST be Neon Serverless PostgreSQL (no local PostgreSQL, no SQLite)
- **NFR-037**: Authentication MUST use Better Auth with JWT (no session-based auth, no OAuth for MVP)
- **NFR-038**: Phase-I CLI code in src/ directory MUST NOT be modified or deleted
- **NFR-039**: Compatible with Python 3.11+ and Node.js 20+

## Out of Scope (Explicitly Forbidden for MVP)

- OAuth/Social login (Google, GitHub, etc.) - JWT only for MVP
- Email verification for registration - direct registration for MVP
- Password reset/forgot password functionality
- User profile management (update email, change password)
- Task sharing or collaboration between users
- Task categories, tags, or labels
- Task due dates or reminders
- Task attachments or file uploads
- Real-time updates via WebSockets (polling or manual refresh acceptable)
- Task search functionality
- Bulk task operations (select multiple, bulk delete)
- Task export (CSV, PDF, etc.)
- Dark mode or theme customization
- Internationalization (i18n) - English only
- Task comments or notes
- Task history/audit log
- Admin dashboard or user management
- API versioning (v1, v2, etc.) - single API version
- Caching layer (Redis) - direct database queries acceptable
- Comprehensive logging and monitoring (basic logging sufficient)
- Deployment scripts or Docker configuration
- Comprehensive test suite (basic tests acceptable)

## Technical Context

**Architecture**: Full-stack web application with decoupled frontend and backend

**Frontend Stack**:
- Framework: Next.js 16+ (React 19+)
- Router: App Router (app/ directory structure)
- Language: TypeScript (strict mode)
- Styling: Tailwind CSS (recommended) or CSS Modules
- HTTP Client: fetch API or axios
- State Management: React hooks (useState, useEffect) and Server Actions

**Backend Stack**:
- Framework: FastAPI (Python 3.11+)
- ORM: SQLModel (built on SQLAlchemy + Pydantic)
- Database: Neon Serverless PostgreSQL
- Authentication: Better Auth with JWT
- Password Hashing: bcrypt or argon2
- Migrations: Alembic

**Database**:
- Provider: Neon Serverless PostgreSQL
- Schema: users, tasks tables with proper foreign keys and indexes
- Connection: Via Neon connection string (environment variable)

**Authentication Flow**:
1. User registers → Backend hashes password → Stores in database → Returns JWT
2. User logs in → Backend validates credentials → Returns JWT
3. Frontend stores JWT in httpOnly cookie
4. Every API request includes JWT → Backend validates → Allows/denies access
5. User logs out → Frontend clears JWT cookie

**API Design**:
- Base URL: http://localhost:8000/api
- Authentication endpoints: POST /auth/register, POST /auth/login
- Task endpoints: GET /tasks, POST /tasks, PUT /tasks/{id}, DELETE /tasks/{id}, PATCH /tasks/{id}/toggle
- All task endpoints require JWT authentication
- OpenAPI docs at /docs

**Deployment Targets** (future, not MVP):
- Frontend: Vercel (Next.js optimized)
- Backend: Railway, Render, or Fly.io
- Database: Neon (already serverless)

**Performance Goals**:
- API response time: < 200ms p95
- Dashboard load time: < 1 second
- Support: 100+ concurrent users

**Scale/Scope**: Multi-user application, expected 100-1000 users, 10-1000 tasks per user

## Project Structure

```
to-do/
├── .specify/                        # Spec-Kit Plus templates (existing)
│   ├── memory/
│   │   └── constitution.md
│   └── templates/
├── history/                         # PHR and ADR records (existing)
│   ├── prompts/
│   └── adr/
├── specs/                           # Feature specifications
│   ├── 001-cli-todo-app/            # Phase-I (existing, do not modify)
│   │   └── spec.md
│   └── 002-phase2-web/              # Phase-II (this spec)
│       └── spec.md                  # This file
├── src/                             # Phase-I CLI code (DO NOT MODIFY)
│   ├── __init__.py
│   ├── models.py
│   ├── todo_service.py
│   └── main.py
├── backend/                         # Phase-II FastAPI backend (to be created)
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                  # FastAPI app entry point
│   │   ├── database.py              # Database connection and session
│   │   ├── models.py                # SQLModel models (User, Task)
│   │   ├── schemas.py               # Pydantic request/response schemas
│   │   ├── auth.py                  # Authentication logic (JWT, hashing)
│   │   ├── dependencies.py          # FastAPI dependencies (get_current_user)
│   │   └── routers/
│   │       ├── __init__.py
│   │       ├── auth.py              # Auth routes (register, login)
│   │       └── tasks.py             # Task CRUD routes
│   ├── alembic/                     # Database migrations
│   │   └── versions/
│   ├── tests/                       # Backend tests (optional for MVP)
│   ├── requirements.txt             # Python dependencies
│   ├── .env.example                 # Environment variables template
│   └── README.md                    # Backend setup instructions
├── frontend/                        # Phase-II Next.js frontend (to be created)
│   ├── app/                         # Next.js App Router
│   │   ├── layout.tsx               # Root layout
│   │   ├── page.tsx                 # Home/landing page (redirect to dashboard)
│   │   ├── register/
│   │   │   └── page.tsx             # Registration page
│   │   ├── login/
│   │   │   └── page.tsx             # Login page
│   │   └── dashboard/
│   │       └── page.tsx             # Dashboard with task list
│   ├── components/                  # React components
│   │   ├── TaskList.tsx
│   │   ├── TaskItem.tsx
│   │   ├── TaskForm.tsx
│   │   └── Navbar.tsx
│   ├── lib/                         # Utility functions
│   │   ├── api.ts                   # API client functions
│   │   └── auth.ts                  # Auth helper functions
│   ├── types/                       # TypeScript types
│   │   └── index.ts
│   ├── middleware.ts                # Route protection middleware
│   ├── public/                      # Static assets
│   ├── package.json
│   ├── tsconfig.json
│   ├── tailwind.config.js
│   ├── .env.example
│   └── README.md                    # Frontend setup instructions
├── README.md                        # Project root README (update for Phase-II)
├── CLAUDE.md                        # AI generation prompts (existing)
└── .gitignore
```

## Example User Journey (Happy Path)

### Registration and First Task

1. User visits http://localhost:3000
2. Clicks "Register" link
3. Enters email: "alice@example.com", password: "SecurePass123!"
4. Submits form → Backend creates user, returns JWT
5. Frontend stores JWT in httpOnly cookie
6. Automatically redirected to /dashboard
7. Dashboard shows "No tasks yet. Create your first task!"
8. Clicks "Add Task" button
9. Fills form:
   - Title: "Buy groceries"
   - Description: "Milk, eggs, bread, coffee"
   - Priority: "high"
10. Submits → Task appears in dashboard with red badge (high priority), checkbox unchecked

### Subsequent Session

1. User visits http://localhost:3000 (next day)
2. Middleware checks for JWT → no valid token found
3. Redirected to /login
4. Enters email: "alice@example.com", password: "SecurePass123!"
5. Submits → Backend validates, returns JWT
6. Redirected to /dashboard
7. Sees previous task "Buy groceries" still there (persisted in database)
8. Clicks checkbox → Task marked complete, title gets strikethrough
9. Clicks "Logout" → JWT cleared, redirected to login page

## Quality Gates Before Next Phase

- ✅ Spec is complete, unambiguous, and covers all user stories with acceptance criteria
- ✅ Authentication flow is fully specified (register, login, logout, JWT handling)
- ✅ User isolation requirements are explicit and testable
- ✅ Security requirements are comprehensive (password hashing, JWT, input validation)
- ✅ Technology stack is specified (Next.js 16+, FastAPI, SQLModel, Neon, Better Auth)
- ✅ All constraints are explicitly stated (Phase-I code untouched, specific versions)
- ✅ Out of scope items are clearly listed to prevent scope creep
- ✅ Data model with relationships is defined (User 1:N Task)
- ✅ API design patterns are specified (RESTful, JWT auth on all endpoints)
- ✅ Project structure is detailed with file organization
- ✅ Edge cases and security considerations are identified
- ✅ Success criteria are measurable and technology-agnostic

## Alignment with Constitution

This specification adheres to the project constitution:

- **Spec-Driven Development**: This is the specification phase of Phase-II following Agentic Dev Stack Workflow
- **Test-First Development**: Acceptance scenarios define expected behavior before implementation
- **Security First**: Comprehensive security requirements (password hashing, JWT, input validation, user isolation)
- **Clean Architecture**: Backend separated into models, schemas, routers, dependencies; frontend separated into pages, components, lib
- **Multi-User Support**: User isolation enforced at database and API level (user_id foreign key, WHERE clauses)
- **Data Persistence**: Neon PostgreSQL for permanent storage (vs Phase-I in-memory)
- **No Phase-I Modification**: Explicit constraint to preserve existing CLI code in src/

**Next Steps**:
1. Review and approve this specification
2. Generate implementation plan (plan.md) using /sp.plan command
3. Generate task breakdown (tasks.md) using /sp.tasks command
4. Execute implementation using /sp.implement command
5. Commit and create PR using /sp.git.commit_pr command

---

**Note to Implementer**: This spec defines WHAT to build, not HOW to build it. The implementation plan (plan.md) will define architectural decisions, and tasks.md will break down specific implementation steps. Follow the Spec-Kit Plus workflow strictly.
