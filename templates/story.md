# Story PRP Template

Tactical template for sprint tasks, user stories, and scoped features.

---

## Story: [TITLE]

> [Original story from Jira/Linear/etc.]

### Metadata
- **Type:** `feature` | `enhancement` | `bug-fix` | `refactor`
- **Complexity:** `small` | `medium` | `large`
- **Affected Systems:** [list components/services]
- **Estimate:** [story points if applicable]

### Acceptance Criteria
- [ ] [Criterion 1 - testable]
- [ ] [Criterion 2 - testable]
- [ ] [Criterion 3 - testable]

---

## Context Discovery

### Auto-Discovered Patterns
_Populated by codebase analysis_

| Pattern | File | Relevance |
|---------|------|-----------|
| [Pattern name] | `path/to/file.py:LINES` | [Why relevant] |
| [Pattern name] | `path/to/file.py:LINES` | [Why relevant] |

### Documentation References
```yaml
documentation:
  - url: "[specific doc URL with anchor]"
    insight: "[Key takeaway]"
  - file: "path/to/internal/doc.md#section"
    insight: "[Key takeaway]"
```

### Known Gotchas
- **[Library/Framework]:** [Specific constraint]
- **[Project-specific]:** [Convention to follow]
- **[Edge case]:** [Handling required]

---

## Implementation Tasks

### Setup Tasks

#### Task 0: READ existing implementations
- **FILES:**
  - `path/to/similar/feature.py` - understand structure
  - `path/to/tests/similar_test.py` - understand test patterns
- **EXTRACT:** Patterns for [specific concern]
- **VALIDATE:** Can describe the pattern in one sentence

---

### Core Tasks

#### Task 1: [ACTION] `path/to/file.py`

- **IMPLEMENT:** [Specific implementation]
- **PATTERN:** `path/to/reference.py:LINE-LINE`
- **IMPORTS:** [Required imports]
- **GOTCHA:** [Watch out for this]
- **VALIDATE:** `ruff check [path] && pytest tests/[path] -v`
- **IF_FAIL:** [Debug hint]

#### Task 2: [ACTION] `path/to/file.py`

- **IMPLEMENT:** [Specific implementation]
- **DEPENDS:** Task 1
- **PATTERN:** `path/to/reference.py:LINE-LINE`
- **VALIDATE:** `[validation command]`
- **IF_FAIL:** [Debug hint]

#### Task 3: [ACTION] `path/to/file.py`

- **IMPLEMENT:** [Specific implementation]
- **DEPENDS:** Task 1, Task 2
- **INTEGRATE:** [How this connects to previous tasks]
- **VALIDATE:** `[validation command]`
- **IF_FAIL:** [Debug hint]

---

### Test Tasks

#### Task N: CREATE `tests/path/to/test_feature.py`

- **IMPLEMENT:** Tests for Tasks 1-3
- **PATTERN:** `tests/similar/test_feature.py`
- **COVER:**
  - Happy path
  - [Edge case 1]
  - [Edge case 2]
- **VALIDATE:** `pytest tests/path/to/test_feature.py -v --cov=src/feature`
- **IF_FAIL:** Check fixtures, verify test database state

---

## Validation Framework

### Level 1 - Syntax & Style
```bash
ruff check [src_paths] && mypy [src_paths] --strict
```

### Level 2 - Unit Tests
```bash
pytest tests/unit/[feature]/ -v --tb=short
```
**Expected:** All pass, >80% coverage on new code

### Level 3 - Integration Tests
```bash
# Start dependencies
docker-compose up -d [services]

# Run integration tests
pytest tests/integration/[feature]/ -v

# Verify endpoints (if API)
curl -s http://localhost:8000/api/[feature]/health | jq '.status'
```
**Expected:** All pass, health returns "ok"

### Level 4 - Domain-Specific
```bash
# [Performance/Security/Contract validation as applicable]
[specific command]
```
**Expected:** [Specific threshold]

---

## Completion Checklist

### Technical
- [ ] All VALIDATE commands pass
- [ ] No new linting errors
- [ ] Type checking passes
- [ ] Tests pass with >80% coverage on new code

### Acceptance
- [ ] [Criterion 1 from above]
- [ ] [Criterion 2 from above]
- [ ] [Criterion 3 from above]

### Quality
- [ ] Follows discovered patterns
- [ ] Gotchas addressed
- [ ] No hardcoded values (config externalized)
- [ ] Error handling matches project conventions

---

## Notes

### Decisions Made
- [Decision 1]: [Rationale]

### Follow-up Items
- [ ] [Future enhancement]
- [ ] [Technical debt to address]

---

# Usage Example

```markdown
## Story: Add password reset functionality

> As a user, I want to reset my password via email so I can regain access to my account.

### Metadata
- **Type:** `feature`
- **Complexity:** `medium`
- **Affected Systems:** auth-service, email-service, frontend
- **Estimate:** 5 points

### Acceptance Criteria
- [ ] User can request password reset with email
- [ ] Reset link expires after 1 hour
- [ ] User can set new password via reset link
- [ ] Old password no longer works after reset

---

## Context Discovery

### Auto-Discovered Patterns

| Pattern | File | Relevance |
|---------|------|-----------|
| Token generation | `src/auth/jwt.py:45-67` | Reuse for reset tokens |
| Email sending | `src/services/email.py:23-45` | Template pattern |
| Auth routes | `src/api/routes/auth.py` | Route structure |

### Documentation References
```yaml
documentation:
  - url: "https://pyjwt.readthedocs.io/en/stable/usage.html#expiration-time-claim"
    insight: "Use exp claim for 1-hour expiry"
  - file: "docs/email-templates.md#transactional"
    insight: "Use PasswordReset template type"
```

### Known Gotchas
- **pyjwt:** Token decode requires same algorithm as encode
- **Email service:** Rate limited to 10/minute per user
- **Security:** Reset token must be single-use (store in DB)

---

## Implementation Tasks

[Tasks would follow the format above...]
```
