# Task PRP Template

Lightweight template for single-file changes, bug fixes, and atomic tasks.

---

## Task: [TITLE]

> One-line description of what this task accomplishes.

### Metadata
- **Type:** `bug-fix` | `enhancement` | `refactor` | `chore`
- **Complexity:** `trivial` | `small` | `medium`
- **Files:** 1-2 files affected

---

## Context

### Primary File
```
path/to/primary/file.py
```

### Pattern Reference
```
path/to/similar/implementation.py:LINE_START-LINE_END
```

### Dependencies
- `package-name==version` (if new)
- `existing-module` from `path/to/module.py`

### Gotchas
- [Non-obvious constraint or quirk]
- [Library-specific behavior to watch for]

---

## Tasks

### Task 1: [ACTION] `path/to/file.py`

**[ACTION]** = `CREATE` | `UPDATE` | `DELETE` | `FIX`

- **IMPLEMENT:** [Specific change in 1-2 sentences]
- **PATTERN:** `path/to/reference.py:45-60`
- **IMPORTS:** [Required imports, comma-separated]
- **VALIDATE:** `[executable validation command]`
- **IF_FAIL:** [Debug hint for common failure]

### Task 2: [ACTION] `path/to/test_file.py` (if needed)

- **IMPLEMENT:** Test for Task 1 changes
- **PATTERN:** `tests/similar_test.py`
- **VALIDATE:** `pytest path/to/test_file.py -v`
- **IF_FAIL:** Check fixture setup, verify imports

---

## Validation

```bash
# Level 1: Syntax
ruff check [target_path] && mypy [target_path]

# Level 2: Unit (if tests added)
pytest [test_path] -v --tb=short
```

---

## Completion Checklist

- [ ] All VALIDATE commands pass
- [ ] No new linting errors introduced
- [ ] Changes follow referenced patterns
- [ ] Gotchas addressed

---

## Notes

_Space for implementation decisions or follow-up items._

---

# Usage Example

```markdown
## Task: Fix user email validation allowing empty strings

> Prevent empty string emails from passing validation in user registration.

### Metadata
- **Type:** `bug-fix`
- **Complexity:** `small`
- **Files:** `src/schemas/user.py`, `tests/schemas/test_user.py`

---

## Context

### Primary File
```
src/schemas/user.py
```

### Pattern Reference
```
src/schemas/product.py:23-35  # Similar field validation
```

### Gotchas
- Pydantic v2: Use `field_validator` not `validator` decorator
- Empty string vs None: Both should fail, but different error messages

---

## Tasks

### Task 1: FIX `src/schemas/user.py`

- **IMPLEMENT:** Add `min_length=1` to email field, add custom validator for format
- **PATTERN:** `src/schemas/product.py:23-35`
- **IMPORTS:** `from pydantic import field_validator`
- **VALIDATE:** `ruff check src/schemas/user.py && mypy src/schemas/user.py`
- **IF_FAIL:** Check Pydantic v2 validator syntax

### Task 2: UPDATE `tests/schemas/test_user.py`

- **IMPLEMENT:** Add test cases for empty string and whitespace-only emails
- **PATTERN:** `tests/schemas/test_product.py:45-60`
- **VALIDATE:** `pytest tests/schemas/test_user.py -v`
- **IF_FAIL:** Check test fixtures, verify ValidationError import

---

## Validation

```bash
ruff check src/schemas/user.py && mypy src/schemas/user.py
pytest tests/schemas/test_user.py -v --tb=short
```

---

## Completion Checklist

- [ ] Empty string email raises ValidationError
- [ ] Whitespace-only email raises ValidationError
- [ ] Valid emails still pass
- [ ] Tests cover both cases
```
