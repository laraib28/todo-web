# Tasks: Phase-II Full-Stack Web Todo Application

**Input**: Design documents from `/specs/002-phase2-web/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/api-spec.md

**Tests**: No automated tests requested in specification - manual testing only

**Organization**: Tasks grouped by user story to enable independent implementation and testing

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Backend**: `backend/app/`, `backend/tests/`
- **Frontend**: `frontend/app/`, `frontend/components/`, `frontend/lib/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and directory structure

- [ ] T001 Create backend directory structure: backend/app/, backend/app/routers/, backend/alembic/, backend/tests/
- [ ] T002 Create frontend directory structure: frontend/app/, frontend/components/, frontend/lib/, frontend/public/
- [ ] T003 [P] Create backend/requirements.txt with FastAPI 0.104+, SQLModel 0.0.14+, python-jose, passlib, uvicorn, psycopg2-binary
- [ ] T004 [P] Create frontend/package.json with Next.js 16+, React 19+, TypeScript 5.3+, Tailwind CSS 3.4+
- [ ] T005 [P] Create backend/.env.example with DATABASE_URL, JWT_SECRET_KEY, CORS_ORIGINS placeholders
- [ ] T006 [P] Create frontend/.env.example with NEXT_PUBLIC_API_URL placeholder
- [ ] T007 [P] Create backend/.gitignore (venv/, __pycache__/, .env, *.pyc)
- [ ] T008 [P] Create frontend/.gitignore (node_modules/, .next/, .env.local, .DS_Store)
- [ ] T009 [P] Create backend/README.md with setup instructions from quickstart.md
- [ ] T010 [P] Create frontend/README.md with setup instructions from quickstart.md

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

### Backend Foundation

- [ ] T011 Create backend/app/__init__.py (empty file for package)
- [ ] T012 Create backend/app/database.py with Neon connection, create_engine, get_session dependency, create_db_and_tables function
- [ ] T013 Create backend/app/models.py with SQLModel base classes (User and Task models with all fields per data-model.md)
- [ ] T014 Create backend/app/schemas.py with Pydantic models: UserRegister, UserLogin, UserResponse, TaskCreate, TaskUpdate, TaskResponse
- [ ] T015 Create backend/app/auth.py with password hashing (bcrypt), JWT creation, JWT decoding functions
- [ ] T016 Create backend/app/dependencies.py with get_current_user dependency (JWT validation and user extraction)
- [ ] T017 Create backend/app/main.py with FastAPI app initialization, CORS middleware, startup event for database tables
- [ ] T018 [P] Create backend/app/routers/__init__.py (empty file for package)
- [ ] T019 Configure CORS in backend/app/main.py with CORS_ORIGINS from environment variable

### Frontend Foundation

- [ ] T020 Create frontend/tsconfig.json with strict mode TypeScript configuration
- [ ] T021 Create frontend/tailwind.config.js with Tailwind CSS configuration
- [ ] T022 Create frontend/next.config.js with Next.js App Router configuration
- [ ] T023 Create frontend/app/layout.tsx with root HTML layout, metadata, and global styles
- [ ] T024 Create frontend/lib/types.ts with TypeScript interfaces: User, Task, ApiError
- [ ] T025 Create frontend/lib/api.ts with base API client (fetch wrapper with credentials: 'include', error handling)
- [ ] T026 Create frontend/middleware.ts with route protection logic (JWT cookie check, redirects for auth/unauth users)
- [ ] T027 Create frontend/app/globals.css with Tailwind directives and base styles

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - User Registration (Priority: P1) 🎯 MVP

**Goal**: Allow new users to create accounts with email and password, receive JWT token, and automatically login

**Independent Test**: Navigate to /register, submit email "test@example.com" and password "SecurePass123!", verify account creation, JWT cookie set, redirect to dashboard

### Backend Implementation for US1

- [ ] T028 [P] [US1] Create POST /api/auth/register endpoint in backend/app/routers/auth.py
- [ ] T029 [US1] Implement user registration logic in auth router: validate email uniqueness, hash password, insert user, generate JWT, set httpOnly cookie
- [ ] T030 [US1] Add validation for password minimum 8 characters in UserRegister schema
- [ ] T031 [US1] Add error handling for duplicate email (409 Conflict) in registration endpoint
- [ ] T032 [US1] Mount auth router to FastAPI app in backend/app/main.py with prefix=/api/auth

### Frontend Implementation for US1

- [ ] T033 [P] [US1] Create frontend/app/register/page.tsx with registration form (email, password fields)
- [ ] T034 [US1] Add client-side validation in registration form (email format, password length minimum 8)
- [ ] T035 [US1] Add register API function in frontend/lib/api.ts (POST /api/auth/register)
- [ ] T036 [US1] Add form submission handler in registration page (call API, handle success/error, redirect to /dashboard)
- [ ] T037 [US1] Add error display in registration form for API errors (duplicate email, validation errors)
- [ ] T038 [P] [US1] Create frontend/components/Navbar.tsx with Register/Login links (initially)

**Checkpoint**: User Story 1 complete - users can register and be automatically logged in with JWT cookie

---

## Phase 4: User Story 2 - User Login (Priority: P1) 🎯 MVP

**Goal**: Allow existing users to authenticate with email/password and receive JWT token

**Independent Test**: Register user (US1), logout, login with same credentials, verify JWT cookie set and redirect to dashboard

### Backend Implementation for US2

- [ ] T039 [P] [US2] Create POST /api/auth/login endpoint in backend/app/routers/auth.py
- [ ] T040 [US2] Implement login logic: verify email exists, verify password with bcrypt, generate JWT, set httpOnly cookie
- [ ] T041 [US2] Return consistent error message "Invalid credentials" for wrong email OR wrong password (prevent user enumeration)
- [ ] T042 [P] [US2] Create POST /api/auth/logout endpoint in backend/app/routers/auth.py (clear JWT cookie)

### Frontend Implementation for US2

- [ ] T043 [P] [US2] Create frontend/app/login/page.tsx with login form (email, password fields)
- [ ] T044 [US2] Add client-side validation in login form (email format, non-empty password)
- [ ] T045 [US2] Add login API function in frontend/lib/api.ts (POST /api/auth/login)
- [ ] T046 [US2] Add form submission handler in login page (call API, handle success/error, redirect to /dashboard)
- [ ] T047 [US2] Add error display in login form for API errors (invalid credentials)
- [ ] T048 [US2] Add logout API function in frontend/lib/api.ts (POST /api/auth/logout)
- [ ] T049 [US2] Add logout handler in Navbar component (call API, redirect to /login)

**Checkpoint**: User Story 2 complete - users can login, logout, and authentication flow is complete

---

## Phase 5: User Story 3 - View Personal Dashboard (Priority: P1) 🎯 MVP

**Goal**: Display user-isolated task list on dashboard, show empty state, enforce authentication

**Independent Test**: Login as User A with tasks, verify only User A's tasks visible; create User B, verify User B sees empty dashboard

### Backend Implementation for US3

- [ ] T050 [P] [US3] Create GET /api/tasks endpoint in backend/app/routers/tasks.py with current_user dependency
- [ ] T051 [US3] Implement get_tasks logic: SELECT tasks WHERE user_id = current_user.id ORDER BY created_at DESC
- [ ] T052 [US3] Mount tasks router to FastAPI app in backend/app/main.py with prefix=/api/tasks
- [ ] T053 [US3] Add 401 Unauthorized response for invalid/missing JWT in get_current_user dependency

### Frontend Implementation for US3

- [ ] T054 [P] [US3] Create frontend/app/dashboard/page.tsx as Server Component with task list display
- [ ] T055 [US3] Add getTasks API function in frontend/lib/api.ts (GET /api/tasks)
- [ ] T056 [US3] Fetch tasks on dashboard load and display in table/card layout with ID, priority, status, title, description
- [ ] T057 [US3] Add empty state UI in dashboard: "No tasks yet. Create your first task!" with Add Task button
- [ ] T058 [US3] Add priority visual indicators in task display (High=red, Medium=yellow, Low=green badges)
- [ ] T059 [US3] Add loading state while fetching tasks
- [ ] T060 [US3] Add error handling for 401 response (redirect to /login with "Please log in to continue")
- [ ] T061 [US3] Update frontend/app/page.tsx to redirect to /dashboard (landing page redirect)
- [ ] T062 [US3] Update Navbar to show Logout button when authenticated (hide Register/Login)

**Checkpoint**: User Story 3 complete - authenticated users can view their task list with user isolation enforced

---

## Phase 6: User Story 4 - Create New Task (Priority: P1) 🎯 MVP

**Goal**: Allow users to create tasks with title, description, priority and see them immediately in dashboard

**Independent Test**: Login, click Add Task, enter title "Buy groceries", description "Milk, eggs", priority "high", verify task appears in dashboard

### Backend Implementation for US4

- [ ] T063 [P] [US4] Create POST /api/tasks endpoint in backend/app/routers/tasks.py with current_user dependency
- [ ] T064 [US4] Implement create_task logic: INSERT task with user_id=current_user.id, title, description, priority (default medium), is_complete=False
- [ ] T065 [US4] Add validation for title non-empty in TaskCreate schema
- [ ] T066 [US4] Add validation for priority enum (high/medium/low) in TaskCreate schema
- [ ] T067 [US4] Return 201 Created with task object on successful creation

### Frontend Implementation for US4

- [ ] T068 [P] [US4] Create frontend/components/TaskForm.tsx with form fields (title, description, priority dropdown)
- [ ] T069 [US4] Add client-side validation in TaskForm (title required, max lengths per spec)
- [ ] T070 [US4] Add createTask API function in frontend/lib/api.ts (POST /api/tasks)
- [ ] T071 [US4] Integrate TaskForm into dashboard page with modal/inline display
- [ ] T072 [US4] Add form submission handler (call API, optimistic UI update, handle success/error)
- [ ] T073 [US4] Add success message display "Task created successfully"
- [ ] T074 [US4] Add error message display for validation errors or API failures
- [ ] T075 [US4] Refresh task list after successful creation (or use optimistic update)

**Checkpoint**: User Story 4 complete - users can create tasks and see them in their dashboard immediately (MVP FEATURE COMPLETE)

---

## Phase 7: User Story 5 - Update Task Details (Priority: P2)

**Goal**: Allow users to edit task title, description, and priority with ownership enforcement

**Independent Test**: Create task "Buy milk", click Edit, change to "Buy almond milk", verify change persists

### Backend Implementation for US5

- [ ] T076 [P] [US5] Create PUT /api/tasks/{task_id} endpoint in backend/app/routers/tasks.py with current_user dependency
- [ ] T077 [US5] Implement update_task logic: SELECT task by ID, verify user_id==current_user.id, UPDATE fields, return updated task
- [ ] T078 [US5] Return 404 Not Found if task doesn't exist
- [ ] T079 [US5] Return 403 Forbidden if task.user_id != current_user.id (ownership check)
- [ ] T080 [US5] Add validation for title non-empty if provided in TaskUpdate schema
- [ ] T081 [US5] Update updated_at timestamp on task modification

### Frontend Implementation for US5

- [ ] T082 [P] [US5] Add edit button to each task in task list with onClick handler
- [ ] T083 [US5] Create edit mode in TaskForm component (pre-fill with existing values)
- [ ] T084 [US5] Add updateTask API function in frontend/lib/api.ts (PUT /api/tasks/{id})
- [ ] T085 [US5] Add update form submission handler (call API, update UI, handle success/error)
- [ ] T086 [US5] Add cancel button in edit mode (discard changes, close form)
- [ ] T087 [US5] Add success message "Task updated successfully"
- [ ] T088 [US5] Handle 403 error (show "Not authorized" message)

**Checkpoint**: User Story 5 complete - users can update their tasks with ownership protection

---

## Phase 8: User Story 6 - Toggle Task Completion (Priority: P2)

**Goal**: Allow users to mark tasks complete/incomplete with single click and visual feedback

**Independent Test**: Create incomplete task, click checkbox, verify task shows complete (strikethrough), click again to mark incomplete

### Backend Implementation for US6

- [ ] T089 [P] [US6] Create PATCH /api/tasks/{task_id}/toggle endpoint in backend/app/routers/tasks.py with current_user dependency
- [ ] T090 [US6] Implement toggle_complete logic: SELECT task, verify ownership, toggle is_complete boolean, UPDATE, return updated task
- [ ] T091 [US6] Return 404 if task doesn't exist
- [ ] T092 [US6] Return 403 if task.user_id != current_user.id
- [ ] T093 [US6] Update updated_at timestamp on toggle

### Frontend Implementation for US6

- [ ] T094 [P] [US6] Add checkbox UI element to each task in task list
- [ ] T095 [US6] Add toggleTask API function in frontend/lib/api.ts (PATCH /api/tasks/{id}/toggle)
- [ ] T096 [US6] Add checkbox onChange handler (call API, optimistic UI update)
- [ ] T097 [US6] Add visual feedback for completed tasks (strikethrough text, checkmark icon)
- [ ] T098 [US6] Add loading state during toggle (disable checkbox, show spinner)
- [ ] T099 [US6] Revert optimistic update if API call fails

**Checkpoint**: User Story 6 complete - users can toggle task completion with immediate visual feedback

---

## Phase 9: User Story 7 - Delete Task (Priority: P3)

**Goal**: Allow users to permanently delete tasks with confirmation modal

**Independent Test**: Create task, click Delete, confirm in modal, verify task removed from dashboard and database

### Backend Implementation for US7

- [ ] T100 [P] [US7] Create DELETE /api/tasks/{task_id} endpoint in backend/app/routers/tasks.py with current_user dependency
- [ ] T101 [US7] Implement delete_task logic: SELECT task, verify ownership, DELETE from database, return 204 No Content or 200 OK
- [ ] T102 [US7] Return 404 if task doesn't exist
- [ ] T103 [US7] Return 403 if task.user_id != current_user.id

### Frontend Implementation for US7

- [ ] T104 [P] [US7] Add delete button to each task in task list
- [ ] T105 [US7] Create confirmation modal component (confirm/cancel buttons)
- [ ] T106 [US7] Add deleteTask API function in frontend/lib/api.ts (DELETE /api/tasks/{id})
- [ ] T107 [US7] Add delete button onClick handler (show confirmation modal)
- [ ] T108 [US7] Add confirm handler in modal (call API, remove from UI, handle success/error)
- [ ] T109 [US7] Add cancel handler (close modal, no action)
- [ ] T110 [US7] Show success message "Task deleted successfully"
- [ ] T111 [US7] Handle 403 error (show "Not authorized" message)

**Checkpoint**: User Story 7 complete - users can delete tasks with confirmation

---

## Phase 10: User Story 8 - User Logout (Priority: P2)

**Goal**: Allow users to logout, clear session, and prevent unauthorized access to protected routes

**Independent Test**: Login, click Logout button, verify redirect to login page, verify /dashboard redirects to /login

### Frontend Implementation for US8

- [ ] T112 [US8] Update logout handler in Navbar (already created in T049) to clear client-side state
- [ ] T113 [US8] Add success message "Successfully logged out" after logout
- [ ] T114 [US8] Verify middleware.ts redirects to /login when access_token cookie is missing
- [ ] T115 [US8] Verify middleware.ts prevents cached dashboard display after logout (test with browser back button)

**Note**: Backend logout endpoint already created in T042 (US2)

**Checkpoint**: User Story 8 complete - users can logout securely with session cleared

---

## Phase 11: User Story 9 - Filter and Sort Tasks (Priority: P4) [OPTIONAL]

**Goal**: Allow users to filter by status (all/complete/incomplete) and sort by priority or date

**Independent Test**: Create 5 tasks (3 incomplete, 2 complete), filter to "Incomplete only", verify only 3 shown

**Note**: This is P4 priority - implement only if time permits after P1-P3 stories are complete

### Frontend Implementation for US9

- [ ] T116 [P] [US9] Add filter dropdown in dashboard (All/Incomplete/Complete)
- [ ] T117 [P] [US9] Add sort dropdown in dashboard (Date/Priority)
- [ ] T118 [US9] Implement filter logic in dashboard (filter task array by is_complete)
- [ ] T119 [US9] Implement sort logic in dashboard (sort by created_at or priority)
- [ ] T120 [US9] Persist filter/sort preferences in localStorage or URL params
- [ ] T121 [US9] Restore filter/sort preferences on page load

**Checkpoint**: User Story 9 complete - users can filter and sort their task list

---

## Phase 12: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T122 [P] Add loading spinner component in frontend/components/LoadingSpinner.tsx
- [ ] T123 [P] Add error message component in frontend/components/ErrorMessage.tsx
- [ ] T124 [P] Add success message component in frontend/components/SuccessMessage.tsx (toast notification)
- [ ] T125 [P] Improve task list UI with better spacing, borders, hover effects in frontend/components/TaskList.tsx
- [ ] T126 [P] Extract TaskItem component from TaskList in frontend/components/TaskItem.tsx
- [ ] T127 Add responsive design for mobile devices (Tailwind breakpoints)
- [ ] T128 Add form field character counters (title max 200, description max 2000)
- [ ] T129 [P] Add security headers middleware in backend/app/main.py (X-Content-Type-Options, X-Frame-Options, X-XSS-Protection)
- [ ] T130 Test complete user journey: register → login → create task → update → toggle → delete → logout
- [ ] T131 Test user isolation: create 2 users, verify tasks are completely separated
- [ ] T132 Test error scenarios: invalid JWT, expired token, wrong credentials, duplicate email
- [ ] T133 [P] Update root README.md with Phase-II setup instructions and architecture overview
- [ ] T134 Verify all environment variables are documented in .env.example files
- [ ] T135 Run quickstart.md validation (follow setup steps on fresh system)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phases 3-11)**: All depend on Foundational phase completion
  - US1 (Registration): Can start after Foundational ✅ Independent
  - US2 (Login): Can start after Foundational ✅ Independent (but requires US1 for testing)
  - US3 (Dashboard): Depends on US1+US2 for authentication flow
  - US4 (Create Task): Depends on US3 for dashboard display
  - US5 (Update Task): Depends on US4 (needs tasks to update)
  - US6 (Toggle Complete): Depends on US4 (needs tasks to toggle)
  - US7 (Delete Task): Depends on US4 (needs tasks to delete)
  - US8 (Logout): Depends on US2 (logout is part of auth flow)
  - US9 (Filter/Sort): Depends on US4 (needs tasks to filter/sort)
- **Polish (Phase 12)**: Depends on all desired user stories being complete

### Recommended Implementation Order

**MVP (Must Have - P1 Stories)**:
1. Phase 1: Setup
2. Phase 2: Foundational
3. Phase 3: US1 (Registration)
4. Phase 4: US2 (Login)
5. Phase 5: US3 (Dashboard)
6. Phase 6: US4 (Create Task)
7. **STOP AND TEST MVP** - Full registration → login → create task → view dashboard flow

**Incremental Additions (P2-P3 Stories)**:
8. Phase 7: US5 (Update Task)
9. Phase 8: US6 (Toggle Complete)
10. Phase 9: US7 (Delete Task)
11. Phase 10: US8 (Logout)
12. **TEST FULL CRUD** - Complete task management lifecycle

**Optional Enhancement (P4)**:
13. Phase 11: US9 (Filter/Sort)

**Final Polish**:
14. Phase 12: Polish & Cross-Cutting

### Within Each User Story

**Backend First Approach**:
1. Create router/endpoint
2. Implement business logic with ownership checks
3. Add validation and error handling
4. Test endpoint via Swagger UI (/docs)

**Then Frontend**:
5. Create UI components
6. Add API client functions
7. Wire up form handlers
8. Add error/success messages
9. Test user flow manually

### Parallel Opportunities

**Setup Phase (Phase 1)**:
- All tasks T003-T010 marked [P] can run in parallel (different files)

**Foundational Phase (Phase 2)**:
- Backend tasks T011-T019 can run in sequence (share files)
- Frontend tasks T020-T027 can run in parallel with backend tasks (different codebases)

**User Stories**:
- Once Foundational complete, multiple developers can work on different stories:
  - Developer A: US1 (Registration) - T028-T038
  - Developer B: US2 (Login) - T039-T049
  - Developer C: US3 (Dashboard) - T050-T062
  - Developer D: US4 (Create Task) - T063-T075
- Later stories (US5-US9) depend on earlier ones for testing context

**Within Each Story**:
- Backend endpoint tasks often marked [P] (different endpoints)
- Frontend component tasks often marked [P] (different components)
- Parallel example for US1:
  - T028 (backend endpoint) parallel with T033 (frontend page)
  - T030-T031 (backend validation) parallel with T034-T037 (frontend validation)

---

## Parallel Example: Foundational Phase

```bash
# Backend team works on these in sequence (shared files):
Task: "Create database.py"
Task: "Create models.py"
Task: "Create auth.py"
...

# Frontend team works on these in parallel (independent):
Task: "Create tsconfig.json"
Task: "Create tailwind.config.js"
Task: "Create next.config.js"
Task: "Create layout.tsx"
Task: "Create types.ts"
Task: "Create api.ts"
Task: "Create middleware.ts"
```

---

## Parallel Example: User Story 1 (Registration)

```bash
# Backend and Frontend can work in parallel:

## Backend Developer:
Task: "Create POST /api/auth/register endpoint"
Task: "Implement registration logic with password hashing"
Task: "Add validation for password minimum 8 characters"
Task: "Add error handling for duplicate email"

## Frontend Developer (parallel):
Task: "Create register/page.tsx with registration form"
Task: "Add client-side validation (email format, password length)"
Task: "Add register API function in api.ts"
Task: "Add form submission handler with error display"
```

---

## Implementation Strategy

### MVP First (P1 Stories Only - Recommended Start)

**Goal**: Get working authentication + task creation ASAP

1. Complete Phase 1: Setup (T001-T010) - ~1 hour
2. Complete Phase 2: Foundational (T011-T027) - ~3-4 hours
3. Complete Phase 3: US1 Registration (T028-T038) - ~2 hours
4. Complete Phase 4: US2 Login (T039-T049) - ~2 hours
5. Complete Phase 5: US3 Dashboard (T050-T062) - ~2-3 hours
6. Complete Phase 6: US4 Create Task (T063-T075) - ~2-3 hours
7. **STOP and VALIDATE MVP**: Test full flow (register → login → create task → view dashboard)
8. Deploy/demo if ready

**MVP Checkpoint**: Users can register, login, and create tasks with full user isolation

### Incremental Delivery (Add P2-P3 Stories)

**After MVP is validated**:

9. Add Phase 7: US5 Update Task (T076-T088) - ~2 hours
10. Add Phase 8: US6 Toggle Complete (T089-T099) - ~1-2 hours
11. Add Phase 9: US7 Delete Task (T100-T111) - ~1-2 hours
12. Add Phase 10: US8 Logout (T112-T115) - ~1 hour
13. **TEST FULL CRUD**: Complete lifecycle (create → update → toggle → delete)
14. Each story adds value without breaking previous stories

### Optional Enhancement (P4)

15. Add Phase 11: US9 Filter/Sort (T116-T121) - ~2-3 hours
16. Only implement if time permits and P1-P3 are solid

### Final Polish

17. Add Phase 12: Polish (T122-T135) - ~2-3 hours
18. UI improvements, testing, documentation

### Parallel Team Strategy

With multiple developers (2-3 people):

**Week 1: Foundation + MVP**
- All: Complete Setup together (Phase 1)
- All: Complete Foundational together (Phase 2) - CRITICAL, don't split
- Split after foundational:
  - Dev A: US1 (Registration) + US3 (Dashboard)
  - Dev B: US2 (Login) + US4 (Create Task)
- Merge and test MVP together

**Week 2: P2-P3 Stories**
- Dev A: US5 (Update) + US7 (Delete)
- Dev B: US6 (Toggle) + US8 (Logout)
- Dev C (if available): US9 (Filter/Sort) or Polish

**Week 3: Polish & Testing**
- All: Phase 12 (Polish), complete testing, deployment

---

## Task Count Summary

- **Phase 1 (Setup)**: 10 tasks (T001-T010)
- **Phase 2 (Foundational)**: 17 tasks (T011-T027)
- **Phase 3 (US1 Registration)**: 11 tasks (T028-T038)
- **Phase 4 (US2 Login)**: 11 tasks (T039-T049)
- **Phase 5 (US3 Dashboard)**: 13 tasks (T050-T062)
- **Phase 6 (US4 Create Task)**: 13 tasks (T063-T075)
- **Phase 7 (US5 Update Task)**: 13 tasks (T076-T088)
- **Phase 8 (US6 Toggle Complete)**: 11 tasks (T089-T099)
- **Phase 9 (US7 Delete Task)**: 12 tasks (T100-T111)
- **Phase 10 (US8 Logout)**: 4 tasks (T112-T115)
- **Phase 11 (US9 Filter/Sort)**: 6 tasks (T116-T121) - OPTIONAL
- **Phase 12 (Polish)**: 14 tasks (T122-T135)

**Total Tasks**: 135 tasks

**MVP Tasks** (P1 stories only): T001-T075 = 75 tasks
**Full Implementation** (P1-P3): T001-T115 = 115 tasks
**With Optional P4**: T001-T121 = 121 tasks
**Complete with Polish**: T001-T135 = 135 tasks

---

## Independent Test Criteria

### User Story 1 (Registration)
- Navigate to /register
- Enter email: "test@example.com", password: "SecurePass123!"
- Submit form
- **Verify**: User created in database, JWT cookie set, redirected to /dashboard
- **Verify**: Duplicate email registration fails with error message

### User Story 2 (Login)
- Register user (use US1 flow)
- Logout
- Navigate to /login
- Enter same credentials
- **Verify**: JWT cookie set, redirected to /dashboard
- **Verify**: Wrong password shows "Invalid credentials"

### User Story 3 (Dashboard)
- Login as User A
- Create some tasks (need US4)
- **Verify**: Dashboard shows only User A's tasks
- Login as User B (different browser/incognito)
- **Verify**: Dashboard shows no tasks for User B (isolation)

### User Story 4 (Create Task)
- Login
- Click "Add Task"
- Enter title: "Buy groceries", description: "Milk, eggs", priority: "high"
- Submit
- **Verify**: Task appears in dashboard immediately
- **Verify**: Task persists after page refresh
- **Verify**: Empty title shows validation error

### User Story 5 (Update Task)
- Create task "Buy milk"
- Click Edit
- Change title to "Buy almond milk"
- Save
- **Verify**: Task updated in dashboard
- **Verify**: Change persists after refresh

### User Story 6 (Toggle Complete)
- Create incomplete task
- Click checkbox
- **Verify**: Task shows complete (strikethrough, checkmark)
- Click checkbox again
- **Verify**: Task shows incomplete

### User Story 7 (Delete Task)
- Create task
- Click Delete
- Confirm in modal
- **Verify**: Task removed from dashboard
- **Verify**: Task deleted from database (refresh, still gone)

### User Story 8 (Logout)
- Login
- Click Logout
- **Verify**: Redirected to /login
- Try to access /dashboard
- **Verify**: Redirected to /login with "Please log in" message

### User Story 9 (Filter/Sort) [OPTIONAL]
- Create 5 tasks (3 incomplete, 2 complete)
- Select filter "Incomplete only"
- **Verify**: Only 3 incomplete tasks shown
- Select sort "Priority (High to Low)"
- **Verify**: High priority tasks shown first

---

## Notes

- [P] tasks = different files, no dependencies, can parallelize
- [Story] label maps task to specific user story for traceability
- Each user story is independently completable and testable
- Stop at any checkpoint to validate story independently
- Commit after each task or logical group of related tasks
- Test via Swagger UI (/docs) for backend endpoints
- Test via browser for frontend flows
- Ensure user isolation is tested thoroughly (2 users, verify complete separation)
- MVP = US1-US4 (75 tasks) delivers core value
- P2-P3 stories (US5-US8) add polish and convenience
- P4 story (US9) is optional enhancement
