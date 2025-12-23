# Task Generation from PRP Template

Generate structured, trackable task lists from completed PRPs.

> Based on ai-dev-tasks workflow (github.com/snarktank/ai-dev-tasks)

---

## Purpose

When a PRP is complete and approved, use this template to:
1. Break it into hierarchical tasks and sub-tasks
2. Create trackable checkbox format
3. List relevant files with rationale
4. Enable incremental implementation with verification

---

## Two-Phase Generation Process

### Phase 1: High-Level Tasks

First, generate parent tasks only. Present for user review before expanding.

```markdown
# Tasks: [Feature Name]

Generated from: `PRPs/[feature-name]-prp.md`
Created: [date]
Status: Planning

## Overview
[One sentence describing the feature goal]

## Parent Tasks

- [ ] **0.0 Create feature branch**
- [ ] **1.0 [First major component]**
- [ ] **2.0 [Second major component]**
- [ ] **3.0 [Third major component]**
- [ ] **4.0 [Testing & validation]**
- [ ] **5.0 [Integration & cleanup]**

---

*Review these parent tasks. Reply "Go" to expand into detailed sub-tasks.*
```

### Phase 2: Expanded Sub-Tasks

After user confirms, expand each parent task:

```markdown
# Tasks: [Feature Name]

Generated from: `PRPs/[feature-name]-prp.md`
Created: [date]
Status: Ready for implementation

## Overview
[One sentence describing the feature goal]

---

## Task 0: Setup

- [ ] **0.0 Create feature branch**
  - [ ] 0.1 Create branch from main: `git checkout -b feature/[name]`
  - [ ] 0.2 Verify clean working directory
  - [ ] 0.3 Update dependencies if needed

---

## Task 1: [First Major Component]

- [ ] **1.0 [Component name]**
  - [ ] 1.1 [First sub-task with specific action]
  - [ ] 1.2 [Second sub-task]
  - [ ] 1.3 [Third sub-task]
  - [ ] 1.4 Validate: `[validation command]`

---

## Task 2: [Second Major Component]

- [ ] **2.0 [Component name]**
  - [ ] 2.1 [Sub-task]
  - [ ] 2.2 [Sub-task]
  - [ ] 2.3 Validate: `[validation command]`

---

## Task 3: [Third Major Component]

- [ ] **3.0 [Component name]**
  - [ ] 3.1 [Sub-task]
  - [ ] 3.2 [Sub-task]
  - [ ] 3.3 Validate: `[validation command]`

---

## Task 4: Testing & Validation

- [ ] **4.0 Comprehensive testing**
  - [ ] 4.1 Run unit tests: `pytest tests/unit/[feature]/ -v`
  - [ ] 4.2 Run integration tests: `pytest tests/integration/[feature]/ -v`
  - [ ] 4.3 Run linting: `ruff check src/[feature]/`
  - [ ] 4.4 Run type check: `mypy src/[feature]/`
  - [ ] 4.5 Verify all acceptance criteria from PRP

---

## Task 5: Integration & Cleanup

- [ ] **5.0 Final integration**
  - [ ] 5.1 Update documentation if needed
  - [ ] 5.2 Review all changes: `git diff main`
  - [ ] 5.3 Create pull request
  - [ ] 5.4 Address review feedback

---

## Relevant Files

| File | Action | Rationale |
|------|--------|-----------|
| `src/[path]/[file].py` | Create | [Why this file] |
| `src/[path]/[file].py` | Modify | [What changes] |
| `tests/[path]/test_[file].py` | Create | Unit tests for [component] |
| `config/[file].yaml` | Modify | [Configuration needed] |

> Note: Unit tests should be placed alongside the code files they test.

---

## Implementation Notes

### From PRP Context
- **Pattern to follow:** `[file:lines from PRP]`
- **Key gotcha:** [From PRP gotchas section]
- **Dependencies:** [From PRP]

### Progress Tracking
After completing each sub-task:
1. Change `- [ ]` to `- [x]`
2. Commit changes if substantial
3. Validate before moving to next task

### If Blocked
- Note the blocker in this file
- Skip to next independent task if possible
- Return to blocked task after resolving

---

## Completion Checklist

- [ ] All sub-tasks marked complete (`[x]`)
- [ ] All validation commands pass
- [ ] Acceptance criteria from PRP satisfied
- [ ] PR created and ready for review
```

---

## Sub-Task Writing Guidelines

### Be Specific and Actionable
```markdown
# Good
- [ ] 1.1 Create UserService class in `src/services/user_service.py` with `get_user(id)` method

# Bad
- [ ] 1.1 Create the user service
```

### Include Validation
```markdown
# Good
- [ ] 1.4 Validate: `pytest tests/unit/test_user_service.py -v`

# Bad
- [ ] 1.4 Make sure it works
```

### Reference PRP Patterns
```markdown
# Good
- [ ] 2.1 Implement auth middleware following pattern in `src/middleware/logging.py:15-40`

# Bad
- [ ] 2.1 Add authentication
```

### One Action Per Sub-Task
```markdown
# Good
- [ ] 3.1 Add email field to User model
- [ ] 3.2 Create migration for email field
- [ ] 3.3 Run migration: `alembic upgrade head`

# Bad
- [ ] 3.1 Add email field to User model and create and run migration
```

---

## Task Numbering Convention

```
X.0  - Parent task (major component)
X.1  - First sub-task
X.2  - Second sub-task
...
X.N  - Validation sub-task (always last)
```

### Standard Parent Tasks

| Number | Purpose | Always Include? |
|--------|---------|-----------------|
| 0.0 | Setup / Branch creation | Yes (unless specified) |
| 1.0-N.0 | Feature components | Based on PRP scope |
| N+1.0 | Testing & Validation | Yes |
| N+2.0 | Integration & Cleanup | Yes |

---

## Mapping PRP Sections to Tasks

| PRP Section | Maps To |
|-------------|---------|
| Implementation Tasks | Parent tasks (1.0, 2.0, etc.) |
| Task IMPLEMENT | Sub-task description |
| Task VALIDATE | Validation sub-task (X.N) |
| Task PATTERN | Implementation notes |
| Task GOTCHA | Implementation notes |
| Success Criteria | Completion checklist |
| Codebase Trees (new files) | Relevant Files table |

---

## Example: PRP to Tasks

### From PRP:
```markdown
### Task 1: CREATE `src/services/auth_service.py`
- IMPLEMENT: AuthService with login, logout, refresh_token methods
- PATTERN: `src/services/user_service.py:20-60`
- IMPORTS: jwt, datetime, UserModel
- GOTCHA: Use RS256 algorithm for production
- VALIDATE: `pytest tests/unit/test_auth_service.py -v`
```

### Generated Task:
```markdown
## Task 1: Authentication Service

- [ ] **1.0 Create AuthService**
  - [ ] 1.1 Create file `src/services/auth_service.py`
  - [ ] 1.2 Implement `login(email, password)` method following `src/services/user_service.py:20-40`
  - [ ] 1.3 Implement `logout(token)` method
  - [ ] 1.4 Implement `refresh_token(token)` method following `src/services/user_service.py:45-60`
  - [ ] 1.5 Add imports: `jwt`, `datetime`, `UserModel`
  - [ ] 1.6 Note: Use RS256 algorithm (not HS256) for production security
  - [ ] 1.7 Validate: `pytest tests/unit/test_auth_service.py -v`
```

---

## Integration with PRP Workflow

```
┌─────────────────┐
│  PRP Creation   │  ← Use prp-builder skill
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  PRP Review &   │  ← Context completeness check
│    Approval     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Task Generation │  ← Use this template
│   (Phase 1)     │     Generate parent tasks
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  User Review    │  ← Approve task structure
│   "Go"          │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Task Expansion  │  ← Expand to sub-tasks
│   (Phase 2)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Implementation  │  ← Work through tasks
│  (Incremental)  │     Check off as complete
└─────────────────┘
```

---

## Output Location

Save generated task files to:
```
tasks/
└── tasks-[feature-name].md
```

Or alongside PRP:
```
PRPs/
├── [feature-name]-prp.md
└── [feature-name]-tasks.md
```
