# Data Model: Phase-II Full-Stack Web Todo Application

**Feature**: 002-phase2-web
**Date**: 2026-01-01
**Database**: Neon Serverless PostgreSQL
**ORM**: SQLModel

## Entity Relationship Diagram

```
┌─────────────────────────┐
│ User                    │
├─────────────────────────┤
│ id (PK)          INT    │
│ email            VARCHAR│ UNIQUE, NOT NULL
│ hashed_password  VARCHAR│ NOT NULL
│ created_at       TIMESTAMP│
└──────────┬──────────────┘
           │
           │ 1:N (one user has many tasks)
           │
           ▼
┌─────────────────────────┐
│ Task                    │
├─────────────────────────┤
│ id (PK)          INT    │
│ user_id (FK)     INT    │ → users.id (CASCADE DELETE)
│ title            VARCHAR│ NOT NULL, MAX 200
│ description      TEXT   │
│ priority         VARCHAR│ DEFAULT 'medium'
│ is_complete      BOOLEAN│ DEFAULT FALSE
│ created_at       TIMESTAMP│
│ updated_at       TIMESTAMP│
└─────────────────────────┘
```

## Entities

### User

**Purpose**: Represents a registered user account with authentication credentials.

**Attributes**:
- `id` (Integer, Primary Key): Auto-incrementing unique identifier
- `email` (String, Unique, Not Null): User's email address for login (indexed for fast lookups)
- `hashed_password` (String, Not Null): bcrypt hashed password (never store plaintext)
- `created_at` (DateTime): Account creation timestamp (UTC)

**Relationships**:
- **One-to-Many with Task**: A user can have multiple tasks (0 to N)
- Foreign key relationship from Task.user_id → User.id

**Indexes**:
- Primary key index on `id` (automatic)
- Unique index on `email` (automatic, enforces uniqueness)

**Constraints**:
- `email` must be unique across all users
- `email` must be valid email format (enforced by Pydantic EmailStr in API layer)
- `hashed_password` must be non-empty

**Security Considerations**:
- Password is hashed with bcrypt (cost factor 12) before storage
- Email used for authentication (username)
- No sensitive PII stored beyond email

**SQLModel Definition**:
```python
from sqlmodel import Field, SQLModel, Relationship
from datetime import datetime

class User(SQLModel, table=True):
    __tablename__ = "users"

    id: int | None = Field(default=None, primary_key=True)
    email: str = Field(unique=True, index=True, max_length=255)
    hashed_password: str = Field(max_length=255)
    created_at: datetime = Field(default_factory=datetime.utcnow)

    # Relationship
    tasks: list["Task"] = Relationship(back_populates="user", cascade_delete=True)
```

---

### Task

**Purpose**: Represents a single todo item belonging to a specific user.

**Attributes**:
- `id` (Integer, Primary Key): Auto-incrementing unique identifier
- `user_id` (Integer, Foreign Key, Not Null): Reference to owning user (users.id)
- `title` (String, Not Null): Task title (max 200 characters)
- `description` (Text, Optional): Detailed task description (max 2000 characters enforced at API)
- `priority` (String, Not Null): Priority level - one of: 'high', 'medium', 'low' (default: 'medium')
- `is_complete` (Boolean, Not Null): Completion status (default: False)
- `created_at` (DateTime): Task creation timestamp (UTC)
- `updated_at` (DateTime): Last modification timestamp (UTC)

**Relationships**:
- **Many-to-One with User**: Each task belongs to exactly one user
- Foreign key constraint on `user_id` referencing `users.id`

**Indexes**:
- Primary key index on `id` (automatic)
- Foreign key index on `user_id` (automatic, for join performance)
- Composite index on `(user_id, created_at)` (recommended for dashboard queries)

**Constraints**:
- `user_id` must reference valid user (foreign key constraint)
- `title` cannot be empty (validated at API layer)
- `priority` must be one of: 'high', 'medium', 'low' (validated at API layer)
- `is_complete` must be boolean (TRUE/FALSE)

**Cascade Rules**:
- **ON DELETE CASCADE**: When user is deleted, all their tasks are automatically deleted
- **ON UPDATE CASCADE**: If user.id changes (unlikely), task.user_id updates automatically

**State Transitions**:
```
┌─────────────┐
│ New Task    │
│ is_complete │
│ = False     │
└──────┬──────┘
       │
       │ User clicks "Mark Complete"
       │
       ▼
┌─────────────┐
│ Complete    │
│ is_complete │
│ = True      │
└──────┬──────┘
       │
       │ User clicks "Mark Incomplete"
       │
       ▼
┌─────────────┐
│ Incomplete  │
│ is_complete │
│ = False     │
└─────────────┘
```

**SQLModel Definition**:
```python
from sqlmodel import Field, SQLModel, Relationship
from datetime import datetime

class Task(SQLModel, table=True):
    __tablename__ = "tasks"

    id: int | None = Field(default=None, primary_key=True)
    user_id: int = Field(foreign_key="users.id", ondelete="CASCADE", index=True)
    title: str = Field(min_length=1, max_length=200)
    description: str = Field(default="", max_length=2000)
    priority: str = Field(default="medium", max_length=10)
    is_complete: bool = Field(default=False)
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    # Relationship
    user: User = Relationship(back_populates="tasks")
```

---

## Database Schema (DDL)

### PostgreSQL Table Definitions

```sql
-- Users table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    hashed_password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Tasks table
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    title VARCHAR(200) NOT NULL,
    description TEXT DEFAULT '',
    priority VARCHAR(10) NOT NULL DEFAULT 'medium',
    is_complete BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

CREATE INDEX idx_tasks_user_id ON tasks(user_id);
CREATE INDEX idx_tasks_user_created ON tasks(user_id, created_at DESC);

-- Trigger to auto-update updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_tasks_updated_at
    BEFORE UPDATE ON tasks
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

**Note**: SQLModel will generate these tables automatically via `SQLModel.metadata.create_all(engine)`. The above SQL is provided for reference and manual inspection.

---

## Validation Rules

### User Entity Validation

**Email**:
- Must be valid email format (validated by Pydantic EmailStr)
- Must be unique (enforced by database constraint)
- Maximum 255 characters
- Case-insensitive for login (convert to lowercase before storage)

**Password (at registration)**:
- Minimum 8 characters (validated at API layer)
- Must be hashed before storage (bcrypt with cost 12)
- Never store plaintext password

**Examples**:
- ✅ Valid: `user@example.com` + password `SecurePass123!`
- ❌ Invalid: `notanemail` (invalid format)
- ❌ Invalid: `user@example.com` (if already exists)
- ❌ Invalid: password `short` (< 8 characters)

### Task Entity Validation

**Title**:
- Required (cannot be empty or whitespace-only)
- Minimum 1 character after trimming
- Maximum 200 characters
- No special validation (allow any UTF-8 characters)

**Description**:
- Optional (can be empty string)
- Maximum 2000 characters (enforced at API layer)
- No special validation

**Priority**:
- Must be one of: `'high'`, `'medium'`, `'low'`
- Case-insensitive (convert to lowercase before storage)
- Default value: `'medium'`

**is_complete**:
- Boolean only (true/false)
- Default value: `false`

**user_id**:
- Must reference existing user
- Cannot be null
- Enforced by foreign key constraint

**Examples**:
- ✅ Valid: title="Buy groceries", description="Milk, eggs", priority="high"
- ✅ Valid: title="Call dentist", description="", priority="medium" (empty description OK)
- ❌ Invalid: title="" (empty title)
- ❌ Invalid: priority="urgent" (not in enum)
- ❌ Invalid: user_id=999 (non-existent user)

---

## Indexing Strategy

### Performance Considerations

**users.email**:
- Unique index (automatic with UNIQUE constraint)
- Critical for login queries: `SELECT * FROM users WHERE email = ?`
- High cardinality (every email is unique)

**tasks.user_id**:
- Non-unique index (automatic with FOREIGN KEY)
- Critical for dashboard queries: `SELECT * FROM tasks WHERE user_id = ?`
- Low-to-medium cardinality (one user has many tasks)

**tasks.(user_id, created_at) COMPOSITE**:
- Recommended for sorted dashboard queries
- Query: `SELECT * FROM tasks WHERE user_id = ? ORDER BY created_at DESC`
- Covers both filtering and sorting in single index lookup

### Query Patterns

**Most Common Queries**:
1. Login: `SELECT * FROM users WHERE email = 'user@example.com'`
   - Uses: `idx_users_email`
2. Get user's tasks: `SELECT * FROM tasks WHERE user_id = 123 ORDER BY created_at DESC`
   - Uses: `idx_tasks_user_created` (composite index)
3. Get specific task: `SELECT * FROM tasks WHERE id = 456 AND user_id = 123`
   - Uses: Primary key + filter (very fast)
4. Update task: `UPDATE tasks SET title = ? WHERE id = 456 AND user_id = 123`
   - Uses: Primary key + filter
5. Delete task: `DELETE FROM tasks WHERE id = 456 AND user_id = 123`
   - Uses: Primary key + filter

---

## Data Integrity

### Foreign Key Constraints

**tasks.user_id → users.id**:
- **ON DELETE CASCADE**: Deleting a user automatically deletes all their tasks
- **ON UPDATE CASCADE**: Updating user.id updates all related task.user_id (rare)

**Rationale**:
- When user account is deleted, their tasks become orphaned and meaningless
- CASCADE delete ensures cleanup and prevents orphaned data
- Maintains referential integrity

### Unique Constraints

**users.email**:
- Prevents duplicate email registrations
- Enforced at database level (not just application)
- Returns error on duplicate insert (caught and handled by API)

### Check Constraints (Future Enhancement)

Optional constraints for additional validation:
```sql
-- Ensure priority is valid enum value
ALTER TABLE tasks ADD CONSTRAINT check_priority
    CHECK (priority IN ('high', 'medium', 'low'));

-- Ensure title is not empty
ALTER TABLE tasks ADD CONSTRAINT check_title_not_empty
    CHECK (LENGTH(TRIM(title)) > 0);
```

**Note**: These are currently enforced at API layer via Pydantic validation. Database-level constraints provide additional safety but are optional for MVP.

---

## Database Migrations

### Migration Strategy

**Tool**: Alembic (recommended for SQLModel/SQLAlchemy)

**Migration Files**:
```
backend/alembic/
├── versions/
│   ├── 001_initial_schema.py     # Create users and tasks tables
│   └── 002_add_indexes.py        # Add composite indexes (if needed later)
├── env.py                        # Alembic environment config
└── alembic.ini                   # Alembic configuration
```

**Initial Migration**:
```bash
# Generate initial migration
alembic revision --autogenerate -m "Initial schema"

# Apply migration
alembic upgrade head
```

**Alternative for MVP**: Use `SQLModel.metadata.create_all(engine)` on startup to auto-create tables. Switch to Alembic when database changes become frequent.

---

## Sample Data

### Example User Record

```json
{
  "id": 1,
  "email": "alice@example.com",
  "hashed_password": "$2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/LewY5NU2uL3zcY9ri",
  "created_at": "2026-01-01T10:30:00Z"
}
```

### Example Task Records

```json
[
  {
    "id": 1,
    "user_id": 1,
    "title": "Buy groceries",
    "description": "Milk, eggs, bread, coffee",
    "priority": "high",
    "is_complete": false,
    "created_at": "2026-01-01T10:35:00Z",
    "updated_at": "2026-01-01T10:35:00Z"
  },
  {
    "id": 2,
    "user_id": 1,
    "title": "Call dentist",
    "description": "Schedule checkup appointment",
    "priority": "medium",
    "is_complete": false,
    "created_at": "2026-01-01T11:00:00Z",
    "updated_at": "2026-01-01T11:00:00Z"
  },
  {
    "id": 3,
    "user_id": 1,
    "title": "Finish project report",
    "description": "",
    "priority": "high",
    "is_complete": true,
    "created_at": "2026-01-01T09:00:00Z",
    "updated_at": "2026-01-01T14:30:00Z"
  }
]
```

---

## User Isolation Enforcement

### Query Patterns for Isolation

**All task queries MUST filter by user_id**:

```python
# ✅ CORRECT: Get tasks for current user only
statement = select(Task).where(Task.user_id == current_user.id)
tasks = session.exec(statement).all()

# ❌ WRONG: Get all tasks (security violation)
statement = select(Task)
tasks = session.exec(statement).all()

# ✅ CORRECT: Get specific task with ownership check
task = session.get(Task, task_id)
if task.user_id != current_user.id:
    raise HTTPException(status_code=403, detail="Not authorized")

# ✅ CORRECT: Update with ownership enforcement
statement = (
    update(Task)
    .where(Task.id == task_id)
    .where(Task.user_id == current_user.id)  # Critical: ensures ownership
    .values(**update_data)
)
session.exec(statement)

# ✅ CORRECT: Delete with ownership enforcement
statement = (
    delete(Task)
    .where(Task.id == task_id)
    .where(Task.user_id == current_user.id)  # Critical: ensures ownership
)
session.exec(statement)
```

### Testing Isolation

**Test Scenario**:
1. Create User A (id=1) and User B (id=2)
2. User A creates Task 1 (user_id=1)
3. User B tries to access Task 1:
   - GET /tasks/1 with User B's JWT → 403 Forbidden
4. User B lists tasks:
   - GET /tasks with User B's JWT → Empty array (Task 1 not visible)
5. User B tries to update Task 1:
   - PUT /tasks/1 with User B's JWT → 403 Forbidden
6. User B tries to delete Task 1:
   - DELETE /tasks/1 with User B's JWT → 403 Forbidden

**Expected Behavior**:
- User B can NEVER see, modify, or delete User A's tasks
- All operations are scoped to authenticated user
- Database enforces user_id foreign key
- API enforces user_id filtering in all queries

---

## Performance Characteristics

### Expected Scale (MVP)

- **Users**: 100-1000 users
- **Tasks per user**: 10-1000 tasks
- **Total tasks**: 10,000-100,000 tasks

### Query Performance

**Login (users.email lookup)**:
- Index: Unique index on email
- Expected: < 10ms
- Scalability: O(log n) with B-tree index

**Dashboard (tasks for user)**:
- Index: Composite index on (user_id, created_at)
- Expected: < 50ms for 1000 tasks
- Scalability: O(log n) for lookup + O(m) for result scan (m = tasks per user)

**Task CRUD operations**:
- Index: Primary key on tasks.id
- Expected: < 10ms
- Scalability: O(1) for primary key lookup

### Optimization Recommendations

**For MVP**:
- ✅ Primary key indexes (automatic)
- ✅ Foreign key indexes (automatic)
- ✅ Unique index on users.email
- ✅ Composite index on (user_id, created_at) for sorted queries

**For Scale (future)**:
- Partitioning by user_id (if > 1M tasks)
- Connection pooling (already configured)
- Read replicas for dashboard queries
- Caching layer (Redis) for frequently accessed users

---

## Security Considerations

### Sensitive Data

**users.hashed_password**:
- Never return in API responses
- Exclude from Pydantic response models
- Use separate UserResponse schema without password

**Example**:
```python
# ❌ WRONG: Exposes hashed password
class UserResponse(BaseModel):
    id: int
    email: str
    hashed_password: str  # NEVER expose this!

# ✅ CORRECT: Password excluded
class UserResponse(BaseModel):
    id: int
    email: str
    created_at: datetime
```

### SQL Injection Prevention

**SQLModel/SQLAlchemy uses parameterized queries**:
```python
# ✅ SAFE: Parameterized query
statement = select(User).where(User.email == email)

# ❌ DANGEROUS: String interpolation (never do this)
query = f"SELECT * FROM users WHERE email = '{email}'"
```

### Mass Assignment Prevention

**Use Pydantic models for input validation**:
```python
# ✅ SAFE: Only specified fields can be updated
class TaskUpdate(BaseModel):
    title: str | None = None
    description: str | None = None
    priority: str | None = None
    # user_id NOT included (prevents user from changing task ownership)
```

---

## Conclusion

This data model provides:
- ✅ Multi-user support with strict isolation
- ✅ Referential integrity via foreign keys
- ✅ Efficient queries via strategic indexing
- ✅ Security through validation and constraints
- ✅ Scalability for expected MVP load (100-1000 users)
- ✅ Clear state management (task completion)
- ✅ Audit trail (created_at, updated_at timestamps)
