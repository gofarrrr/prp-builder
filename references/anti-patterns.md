# PRP Anti-Patterns

What to avoid when creating PRPs. Each anti-pattern includes detection signs and fixes.

---

## 1. Vague Context References

**Anti-Pattern:**
```markdown
## Context
- Check the authentication module
- Follow existing patterns
- Use our standard approach
```

**Problem:** Forces the agent to spend tokens discovering what you already know.

**Fix:**
```markdown
## Context
- File: `src/auth/jwt_handler.py:45-78`
- Pattern: Follow `src/auth/oauth.py` token refresh logic
- Standard: Use `BaseAuthService` from `src/services/base.py`
```

**Detection:** If you can't point to a specific file:line, the reference is too vague.

---

## 2. Implicit Dependencies

**Anti-Pattern:**
```markdown
## Tasks
1. Create the user model
2. Add the API endpoint
3. Write tests
```

**Problem:** No explicit ordering. Agent may attempt tasks in wrong order.

**Fix:**
```markdown
## Tasks
### Task 1: CREATE `models/user.py` [No dependencies]
### Task 2: CREATE `api/users.py` [Depends: Task 1]
### Task 3: CREATE `tests/test_users.py` [Depends: Task 1, 2]
```

**Detection:** Ask "could task N be done before task N-1?" If unclear, dependencies are implicit.

---

## 3. Missing Validation

**Anti-Pattern:**
```markdown
### Task 2: Add authentication middleware
- Implement JWT verification
- Add to route handlers
```

**Problem:** No way to verify completion. Silent failures propagate.

**Fix:**
```markdown
### Task 2: ADD `middleware/auth.py`
- IMPLEMENT: JWT verification using `python-jose`
- INTEGRATE: Add to `main.py` app middleware
- VALIDATE: `pytest tests/test_auth_middleware.py -v`
- IF_FAIL: Check JWT_SECRET env var, verify token format
```

**Detection:** If a task has no executable validation command, it's incomplete.

---

## 4. Over-Specified Implementation

**Anti-Pattern:**
```markdown
### Task 1: Create user service
```python
class UserService:
    def __init__(self, db: Database):
        self.db = db
        self._cache = {}

    async def get_user(self, user_id: int) -> User:
        if user_id in self._cache:
            return self._cache[user_id]
        user = await self.db.users.get(user_id)
        self._cache[user_id] = user
        return user
    # ... 50 more lines
```
```

**Problem:** Removes agent's ability to adapt. Brittle if codebase differs.

**Fix:**
```markdown
### Task 1: CREATE `services/user_service.py`
- IMPLEMENT: UserService with get_user, create_user methods
- PATTERN: Follow `services/auth_service.py` structure
- INTEGRATE: Use existing `Database` from `db/connection.py`
- GOTCHA: Include caching similar to `services/product_service.py:34`
```

**Detection:** If you're writing full implementations, you should just write the code directly.

---

## 5. Context Overload

**Anti-Pattern:**
```markdown
## Documentation References
- FastAPI complete documentation
- SQLAlchemy ORM guide
- Pydantic v2 migration guide
- Python asyncio documentation
- Our entire API design guide
- Authentication best practices (50 pages)
```

**Problem:** Consumes context budget with low-signal information.

**Fix:**
```markdown
## Documentation References
documentation:
  - url: "https://fastapi.tiangolo.com/tutorial/dependencies/#dependencies-with-yield"
    insight: "Use yield for DB session management"
  - file: "docs/api-conventions.md#error-responses"
    insight: "All errors return {error: string, code: int}"
```

**Detection:** If you're linking entire docs instead of specific sections, it's overload.

---

## 6. Missing Gotchas

**Anti-Pattern:**
```markdown
### Task 3: Add database migration
- Create alembic migration for users table
- Run migration
```

**Problem:** Known issues aren't documented. Agent will hit the same problems you did.

**Fix:**
```markdown
### Task 3: ADD migration `migrations/versions/xxx_add_users.py`
- CREATE: users table with id, email, hashed_password, created_at
- GOTCHA: Use `sa.Text` not `sa.String` for email (our Postgres config)
- GOTCHA: Add index on email - see `migrations/versions/001_products.py:23`
- VALIDATE: `alembic upgrade head && alembic downgrade -1 && alembic upgrade head`
```

**Detection:** If you learned something non-obvious implementing similar features, document it.

---

## 7. Scope Creep in Tasks

**Anti-Pattern:**
```markdown
### Task 2: Add user authentication
- Implement login endpoint
- Add registration
- Password reset flow
- Email verification
- OAuth integration
- Session management
- Rate limiting
- Audit logging
```

**Problem:** Single task spans multiple concerns. Too large to validate atomically.

**Fix:**
```markdown
### Task 2: ADD `api/auth/login.py`
- IMPLEMENT: POST /auth/login with email/password
- PATTERN: Follow `api/auth/logout.py` response format
- VALIDATE: `pytest tests/api/test_login.py -v`

### Task 3: ADD `api/auth/register.py`
- IMPLEMENT: POST /auth/register
- DEPENDS: Task 2 (uses same auth utilities)
- VALIDATE: `pytest tests/api/test_register.py -v`

[Continue with separate tasks for each concern]
```

**Detection:** If a task has more than 3-4 bullet points, it should probably be split.

---

## 8. Ignoring Existing Patterns

**Anti-Pattern:**
```markdown
### Task 1: Create new API endpoint
- Use Flask-style routing
- Return XML responses
- Custom error format
```

**Problem:** Introduces inconsistency. Ignores established conventions.

**Fix:**
```markdown
### Task 1: CREATE `api/routes/new_feature.py`
- PATTERN: Follow `api/routes/users.py` structure exactly
- RESPONSE: Use `ResponseModel` from `api/schemas/base.py`
- ERRORS: Use `raise_http_exception()` from `api/utils/errors.py`
- VALIDATE: Response matches existing endpoint format
```

**Detection:** Before creating new patterns, search for existing ones. If found, reference them.

---

## 9. No Rollback Strategy

**Anti-Pattern:**
```markdown
### Task 5: Deploy to production
- Run database migration
- Deploy new code
- Update configuration
```

**Problem:** No recovery path if something fails mid-deployment.

**Fix:**
```markdown
### Task 5: Deploy sequence
- PRE-CHECK: `alembic current` - note current version
- BACKUP: `pg_dump $DATABASE_URL > backup_$(date +%Y%m%d).sql`
- MIGRATE: `alembic upgrade head`
- IF_FAIL: `alembic downgrade [noted_version]`
- DEPLOY: `kubectl apply -f k8s/deployment.yaml`
- VERIFY: `curl -s https://api.example.com/health | jq '.version'`
- ROLLBACK: `kubectl rollout undo deployment/api`
```

**Detection:** For any destructive or stateful operation, ask "how do we undo this?"

---

## 10. Copy-Paste Documentation

**Anti-Pattern:**
```markdown
## FastAPI Dependency Injection

FastAPI provides a powerful dependency injection system. Dependencies are
declared using the `Depends` function. You can use dependencies to:
- Share logic between endpoints
- Enforce security
- Connect to databases
[... 500 more words copied from docs ...]
```

**Problem:** Wastes context on information the model already knows or can retrieve.

**Fix:**
```markdown
## Dependency Pattern
- USE: `Depends(get_db)` from `api/deps.py` for database sessions
- NOTE: Our deps use `yield` pattern - see `api/deps.py:12-25`
```

**Detection:** If content is from external docs and not project-specific, delete it.

---

## Quick Reference Checklist

Before finalizing a PRP, verify:

| Anti-Pattern | Check |
|--------------|-------|
| Vague references | Every reference has file:line? |
| Implicit dependencies | Task order explicit? |
| Missing validation | Every task has VALIDATE? |
| Over-specified | Patterns referenced, not copied? |
| Context overload | Only specific sections linked? |
| Missing gotchas | Non-obvious learnings documented? |
| Scope creep | Each task 3-4 items max? |
| Ignoring patterns | Existing code searched first? |
| No rollback | Destructive ops have recovery? |
| Copy-paste docs | Only project-specific context? |
