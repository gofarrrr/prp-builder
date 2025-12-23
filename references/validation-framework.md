# Four-Level Validation Framework

Catch issues at the right stage. Each level builds on the previous.

---

## Level 1: Syntax & Style

**Purpose:** Catch surface-level issues before deeper testing.

**Tools:**
```bash
# Python
ruff check src/
ruff format --check src/
mypy src/ --strict

# TypeScript
eslint src/
tsc --noEmit

# General
prettier --check "src/**/*.{ts,tsx,json}"
```

**When to Run:** After every file change, before committing.

**What It Catches:**
- Import errors
- Type mismatches
- Formatting inconsistencies
- Unused variables
- Basic syntax errors

**PRP Format:**
```markdown
### Validation Level 1
```bash
ruff check [target_path] && mypy [target_path]
```
```

---

## Level 2: Unit Tests

**Purpose:** Verify individual components work in isolation.

**Tools:**
```bash
# Python
pytest tests/unit/test_[feature].py -v
pytest --cov=[module] --cov-report=term-missing

# TypeScript/JavaScript
jest --testPathPattern="[feature]"
vitest run [feature].test.ts
```

**When to Run:** After completing each implementation task.

**What It Catches:**
- Logic errors in functions
- Edge case handling
- Return value correctness
- Error handling paths

**PRP Format:**
```markdown
### Validation Level 2
```bash
pytest tests/unit/test_auth.py -v --tb=short
```
Expected: All tests pass, >80% coverage on new code
```

---

## Level 3: Integration Tests

**Purpose:** Verify components work together correctly.

**Tools:**
```bash
# API/Service testing
pytest tests/integration/ -v
curl -X GET http://localhost:8000/health

# Database validation
psql -c "SELECT COUNT(*) FROM [new_table]"

# End-to-end
playwright test e2e/[feature].spec.ts
```

**When to Run:** After all tasks in a story/feature are complete.

**What It Catches:**
- Database connection issues
- API contract violations
- Service communication failures
- Configuration problems
- Race conditions

**PRP Format:**
```markdown
### Validation Level 3
```bash
# Start services
docker-compose up -d

# Run integration tests
pytest tests/integration/test_auth_flow.py -v

# Verify endpoints
curl -s http://localhost:8000/api/v1/auth/health | jq '.status'
# Expected: "ok"

# Check database
psql $DATABASE_URL -c "SELECT COUNT(*) FROM users WHERE created_at > NOW() - INTERVAL '1 hour'"
```
```

---

## Level 4: Domain-Specific Validation

**Purpose:** Verify domain requirements and non-functional requirements.

**Categories:**

### Performance Testing
```bash
# Load testing
k6 run scripts/load_test.js --vus 100 --duration 30s

# Benchmark
pytest tests/benchmarks/ --benchmark-only

# Memory profiling
python -m memory_profiler src/heavy_operation.py
```

### Security Scanning
```bash
# Dependency vulnerabilities
pip-audit
npm audit

# Static analysis
bandit -r src/
semgrep --config auto src/

# Secrets detection
gitleaks detect --source .
```

### API Contract Validation
```bash
# OpenAPI validation
openapi-spec-validator openapi.yaml

# Schema compliance
ajv validate -s schema.json -d response.json
```

### MCP Server Validation (for agent tools)
```bash
# Tool schema validation
mcp validate-tool src/tools/new_tool.py

# Integration test
mcp test-server --tool new_tool --input '{"param": "value"}'
```

**PRP Format:**
```markdown
### Validation Level 4
```bash
# Performance baseline
pytest tests/benchmarks/test_auth_performance.py --benchmark-only
# Expected: p95 < 100ms

# Security scan
bandit -r src/auth/ -ll
# Expected: No high-severity issues

# Contract validation
openapi-spec-validator docs/openapi.yaml
# Expected: Valid schema
```
```

---

## Validation Placement in PRPs

### Task PRP
```markdown
## Task 1: CREATE `src/auth/jwt.py`
- IMPLEMENT: JWT token generation
- VALIDATE: `ruff check src/auth/jwt.py && pytest tests/unit/test_jwt.py -v`
- IF_FAIL: Check import paths, verify SECRET_KEY in env
```

### Story PRP
```markdown
## Implementation Tasks

### Task 1: Create JWT utilities
...
VALIDATE: Level 1 + Level 2

### Task 2: Add auth routes
...
VALIDATE: Level 1 + Level 2

### Task 3: Integration
...
VALIDATE: Level 3

## Final Validation
Run all four levels before marking complete.
```

### Base PRP
```markdown
## Validation Framework

### Level 1 - Syntax & Style
[Full commands for entire feature scope]

### Level 2 - Unit Tests
[Component-by-component test commands]

### Level 3 - Integration Tests
[Service startup, API tests, DB validation]

### Level 4 - Domain-Specific
[Performance, security, contracts as applicable]
```

---

## Common Validation Failures & Fixes

| Failure | Likely Cause | Fix |
|---------|--------------|-----|
| Import error | Missing dependency | Add to requirements.txt/package.json |
| Type error | Schema mismatch | Check Pydantic/TypeScript definitions |
| Test timeout | Async issue | Add proper await/async handling |
| Integration fail | Service not running | Check docker-compose, env vars |
| Coverage drop | Untested branch | Add edge case tests |

---

## Anti-Patterns

**Don't:**
- Skip Level 1 validation ("it's just formatting")
- Run only Level 2 ("unit tests pass, ship it")
- Save all validation for the end
- Use vague validation ("make sure it works")

**Do:**
- Validate immediately after each task
- Include specific expected outcomes
- Provide IF_FAIL debug hints
- Run full validation before completion
