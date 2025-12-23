# Base PRP Template

Comprehensive template for major features, new systems, and architectural work.

---

## Feature: [TITLE]

> [One paragraph executive summary]

---

## Goal

### Feature Goal
[Specific, measurable end state - what exists when this is done]

### Deliverable
[Concrete artifact type: API endpoints, UI components, service, etc.]

### Success Definition
[How we know it's complete - observable, testable outcomes]

---

## User Persona

### Target User
[Role/type of user who benefits]

### Primary Use Case
[The main scenario this enables]

### User Workflow
1. User [action]
2. System [response]
3. User [next action]
4. System [outcome]

### Pain Points Addressed
- [Current pain point 1] → [How feature solves it]
- [Current pain point 2] → [How feature solves it]

---

## Why

### Business Value
[Why this matters to the business]

### Problem Statement
[What problem exists without this feature]

### Integration Context
[How this fits with existing systems]

---

## What

### User-Visible Behavior
- [Behavior 1]
- [Behavior 2]
- [Behavior 3]

### Technical Requirements
- [Requirement 1 with constraints]
- [Requirement 2 with constraints]
- [Requirement 3 with constraints]

### Success Criteria (Checkboxes)
- [ ] [Measurable criterion 1]
- [ ] [Measurable criterion 2]
- [ ] [Measurable criterion 3]
- [ ] [Measurable criterion 4]

---

## Context Completeness Check

> Could another developer implement this using ONLY this PRP?

| Requirement | Specified | Location |
|-------------|-----------|----------|
| All file paths | ☐ | |
| Library versions | ☐ | |
| Pattern references | ☐ | |
| API contracts | ☐ | |
| Database schemas | ☐ | |
| Configuration | ☐ | |
| Gotchas documented | ☐ | |
| Validation commands | ☐ | |

---

## Documentation & References

```yaml
documentation:
  external:
    - url: "[URL with section anchor]"
      insight: "[Specific takeaway]"
    - url: "[URL with section anchor]"
      insight: "[Specific takeaway]"

  internal:
    - file: "path/to/doc.md#section"
      insight: "[Specific takeaway]"
    - file: "path/to/doc.md#section"
      insight: "[Specific takeaway]"

  patterns:
    - file: "path/to/file.py:LINE-LINE"
      description: "[What pattern this demonstrates]"
    - file: "path/to/file.py:LINE-LINE"
      description: "[What pattern this demonstrates]"
```

---

## Codebase Trees

### Current State
```
project/
├── src/
│   ├── services/
│   │   └── existing_service.py
│   ├── api/
│   │   └── routes/
│   │       └── existing_route.py
│   └── models/
│       └── existing_model.py
└── tests/
    └── [existing test structure]
```

### Desired State
```
project/
├── src/
│   ├── services/
│   │   ├── existing_service.py
│   │   └── new_service.py          # NEW
│   ├── api/
│   │   └── routes/
│   │       ├── existing_route.py
│   │       └── new_route.py        # NEW
│   ├── models/
│   │   ├── existing_model.py
│   │   └── new_model.py            # NEW
│   └── schemas/
│       └── new_schema.py           # NEW
└── tests/
    ├── unit/
    │   └── test_new_service.py     # NEW
    └── integration/
        └── test_new_feature.py     # NEW
```

---

## Known Gotchas

### Library Constraints
```
[library-name]:
  - Constraint: [specific limitation]
  - Workaround: [how to handle]
  - Reference: [doc link or file:line]

[library-name]:
  - Constraint: [specific limitation]
  - Workaround: [how to handle]
```

### Project-Specific
```
[component]:
  - Rule: [convention to follow]
  - Example: path/to/example.py:LINE

[component]:
  - Rule: [convention to follow]
  - Example: path/to/example.py:LINE
```

---

## Data Models

### ORM Models
```python
# path/to/models/new_model.py
class NewModel(Base):
    __tablename__ = "new_table"

    id: Mapped[int] = mapped_column(primary_key=True)
    # [other fields with types]
```

### Pydantic Schemas
```python
# path/to/schemas/new_schema.py
class NewSchema(BaseModel):
    field: type
    # [validation rules]
```

### Migrations
```sql
-- migrations/versions/XXX_description.py
CREATE TABLE new_table (
    id SERIAL PRIMARY KEY,
    -- [other columns]
);
```

---

## Implementation Tasks

### Phase 1: Foundation

#### Task 1: CREATE models
- **FILE:** `src/models/new_model.py`
- **IMPLEMENT:** ORM model as specified above
- **PATTERN:** `src/models/existing_model.py`
- **VALIDATE:** `mypy src/models/new_model.py`
- **IF_FAIL:** Check SQLAlchemy 2.0 Mapped syntax

#### Task 2: CREATE schemas
- **FILE:** `src/schemas/new_schema.py`
- **IMPLEMENT:** Pydantic models for request/response
- **PATTERN:** `src/schemas/existing_schema.py`
- **DEPENDS:** Task 1
- **VALIDATE:** `ruff check src/schemas/ && mypy src/schemas/`
- **IF_FAIL:** Check Pydantic v2 syntax

### Phase 2: Business Logic

#### Task 3: CREATE service
- **FILE:** `src/services/new_service.py`
- **IMPLEMENT:** Core business logic
- **PATTERN:** `src/services/existing_service.py:20-80`
- **DEPENDS:** Task 1, 2
- **IMPORTS:** [List required imports]
- **GOTCHA:** [Service-specific constraint]
- **VALIDATE:** `pytest tests/unit/test_new_service.py -v`
- **IF_FAIL:** Check database session handling

#### Task 4: CREATE unit tests
- **FILE:** `tests/unit/test_new_service.py`
- **IMPLEMENT:** Tests for service methods
- **PATTERN:** `tests/unit/test_existing_service.py`
- **DEPENDS:** Task 3
- **COVER:** Happy path, edge cases, error handling
- **VALIDATE:** `pytest tests/unit/test_new_service.py -v --cov`
- **IF_FAIL:** Check fixture setup

### Phase 3: API Layer

#### Task 5: CREATE routes
- **FILE:** `src/api/routes/new_route.py`
- **IMPLEMENT:** API endpoints
- **PATTERN:** `src/api/routes/existing_route.py`
- **DEPENDS:** Task 3
- **REGISTER:** Add to `src/api/router.py`
- **VALIDATE:** `curl localhost:8000/api/new-feature/health`
- **IF_FAIL:** Check route registration, verify server running

#### Task 6: CREATE integration tests
- **FILE:** `tests/integration/test_new_feature.py`
- **IMPLEMENT:** End-to-end API tests
- **PATTERN:** `tests/integration/test_existing.py`
- **DEPENDS:** Task 5
- **VALIDATE:** `pytest tests/integration/test_new_feature.py -v`
- **IF_FAIL:** Check test database, verify fixtures

---

## Implementation Patterns

### Critical Pattern 1: [Name]
```python
# Inline comments explain non-obvious requirements
def example_pattern():
    # GOTCHA: Must do X before Y because [reason]
    step_x()

    # Pattern from existing_file.py:45
    step_y()
```

### Critical Pattern 2: [Name]
```python
# Another pattern with gotcha comments
```

---

## Integration Points

### Database Migrations
```bash
# Generate migration
alembic revision --autogenerate -m "add_new_feature"

# Verify migration
alembic upgrade head
alembic downgrade -1
alembic upgrade head
```

### Configuration
```python
# Add to src/config/settings.py
NEW_FEATURE_ENABLED: bool = True
NEW_FEATURE_LIMIT: int = 100
```

### Route Registration
```python
# Add to src/api/router.py
from src.api.routes.new_route import router as new_router
app.include_router(new_router, prefix="/api/new-feature", tags=["new-feature"])
```

---

## Validation Framework

### Level 1 - Syntax & Style
```bash
ruff check src/[feature]/ --fix
ruff format src/[feature]/
mypy src/[feature]/ --strict
```
**Expected:** 0 errors

### Level 2 - Unit Tests
```bash
pytest tests/unit/[feature]/ -v --tb=short
pytest tests/unit/[feature]/ --cov=src/[feature] --cov-report=term-missing
```
**Expected:** All pass, >80% coverage

### Level 3 - Integration Tests
```bash
# Start services
docker-compose up -d

# Wait for ready
./scripts/wait-for-it.sh localhost:8000

# Run tests
pytest tests/integration/[feature]/ -v

# Verify health
curl -s http://localhost:8000/api/[feature]/health | jq '.status'
```
**Expected:** All pass, health returns "ok"

### Level 4 - Domain-Specific
```bash
# Performance
pytest tests/benchmarks/[feature]/ --benchmark-only
# Expected: p95 < [threshold]

# Security
bandit -r src/[feature]/ -ll
# Expected: No high-severity issues

# API Contract
openapi-spec-validator docs/openapi.yaml
# Expected: Valid
```

---

## Completion Artifacts

### Final Validation Checklist

#### Technical
- [ ] Level 1 validation passes
- [ ] Level 2 validation passes (>80% coverage)
- [ ] Level 3 validation passes
- [ ] Level 4 validation passes

#### Feature Success
- [ ] [Success criterion 1 from above]
- [ ] [Success criterion 2 from above]
- [ ] [Success criterion 3 from above]

#### Code Quality
- [ ] Follows patterns from referenced files
- [ ] Gotchas addressed
- [ ] No hardcoded configuration
- [ ] Error handling consistent with project

#### Documentation
- [ ] OpenAPI spec updated
- [ ] README updated if needed
- [ ] Migration documented

---

## Anti-Patterns to Avoid

1. **Don't** create new patterns when existing ones apply
2. **Don't** skip validation between tasks
3. **Don't** hardcode configuration values
4. **Don't** swallow exceptions without logging
5. **Don't** add dependencies without documenting
6. **Don't** implement beyond specified scope
7. **Don't** skip integration tests for "simple" features
8. **Don't** ignore type hints on public interfaces
