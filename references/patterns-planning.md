# Planning Design Patterns

Patterns for structured reasoning, planning, and self-improvement.

---

## Overview

Planning patterns enable agents to reason about tasks before executing, reflect on results, and improve iteratively.

---

## 1. Plan-and-Execute

**What:** Create explicit plan before taking any action, then execute step-by-step.

**When to Use:**
- Complex, multi-step tasks
- Execution is costly/irreversible
- Need visibility into approach

**Pattern:**
```
Task
  │
  ▼
┌─────────────┐
│   PLAN      │
│  (think)    │ ← Generate steps
└──────┬──────┘
       │
       ▼
  [Step 1, Step 2, Step 3]
       │
       ▼
┌─────────────┐
│  EXECUTE    │
│  (act)      │ ← Follow plan
└──────┬──────┘
       │
       ▼
    Result
```

**PRP Application:**
```markdown
## Feature Implementation (Plan-and-Execute)

### Planning Phase
Before any code:
1. Analyze requirements
2. Identify affected files
3. Determine implementation order
4. Define validation criteria
5. Document rollback strategy

### Plan Output
```yaml
plan:
  steps:
    - id: 1
      action: "Create auth_service.py"
      validate: "file exists, imports resolve"

    - id: 2
      action: "Implement login method"
      validate: "unit test passes"

    - id: 3
      action: "Add JWT middleware"
      validate: "integration test passes"

  rollback: "git stash && git checkout ."
```

### Execution Phase
Follow plan step-by-step:
- Execute step 1
- Validate step 1
- IF pass → Execute step 2
- IF fail → Rollback, revise plan
```

---

## 2. ReAct (Reasoning + Acting)

**What:** Interleave reasoning and action in a loop until task complete.

**When to Use:**
- Dynamic tasks requiring adaptation
- Information discovered during execution
- Exploration needed

**Pattern:**
```
┌──────────────────────────────────┐
│                                  │
│  ┌─────────┐    ┌─────────┐     │
│  │ REASON  │───►│   ACT   │     │
│  │ (think) │    │  (do)   │     │
│  └────▲────┘    └────┬────┘     │
│       │              │          │
│       │    ┌─────────▼──────┐   │
│       │    │    OBSERVE     │   │
│       └────│   (perceive)   │   │
│            └────────────────┘   │
│                                  │
└──────────────────────────────────┘
         │
    (loop until done)
         │
         ▼
      Result
```

**ReAct Trace Example:**
```
Thought: I need to find existing service patterns
Action: grep "class.*Service" src/
Observation: Found 3 services in src/services/

Thought: UserService looks most relevant
Action: read src/services/user_service.py
Observation: Uses BaseService, dependency injection

Thought: I have the pattern, now create auth service
Action: write src/services/auth_service.py
Observation: File created successfully

Thought: Need to validate with tests
Action: pytest tests/unit/test_auth.py -v
Observation: All tests pass

Thought: Task complete, all criteria met
Action: [FINISH]
```

**PRP Application:**
```markdown
## Dynamic Discovery (ReAct)

### Initial State
- Goal: Find and follow existing patterns
- Known: Project uses Python + FastAPI
- Unknown: Specific patterns, file locations

### ReAct Loop
```yaml
loop:
  max_iterations: 10
  stop_condition: "all patterns discovered"

  reasoning_prompt: |
    Based on observations so far, what do I need to find next?
    What action will get me that information?

  action_space:
    - glob: "Find files by pattern"
    - grep: "Search file contents"
    - read: "Read specific file"
    - ask: "Ask user for clarification"
```

### Termination
- SUCCESS: All required patterns found
- FAIL: Max iterations reached without success
- ESCALATE: User clarification needed
```

---

## 3. Reflection

**What:** Agent critiques its own output and improves it.

**When to Use:**
- Quality is critical
- Self-improvement possible
- Time for iteration

**Pattern:**
```
Task
  │
  ▼
┌─────────────┐
│  GENERATE   │
│  (draft)    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  REFLECT    │
│  (critique) │
└──────┬──────┘
       │
   ┌───┴───┐
   │       │
 Good    Needs Work
   │       │
   ▼       ▼
Output   Revise ──► Loop
```

**PRP Application:**
```markdown
## PRP Quality Reflection

### Generation Phase
Generate initial PRP draft

### Reflection Prompt
```
Review this PRP draft against quality criteria:

1. Context Completeness
   - Are all file references specific (file:line)?
   - Are patterns referenced, not described?

2. Validation Coverage
   - Does every task have a VALIDATE command?
   - Are IF_FAIL hints provided?

3. Scope Appropriateness
   - Is scope minimal and focused?
   - Are irrelevant systems excluded?

4. Agent-Readiness
   - Could another agent implement from this alone?
   - Are all assumptions explicit?

List specific issues and suggest fixes.
```

### Revision Loop
```yaml
reflection_loop:
  max_iterations: 3

  continue_if: "reflection found issues"
  stop_if: "reflection approves OR max iterations"

  revision_prompt: |
    Address the following issues:
    {reflection_issues}

    Generate improved version.
```
```

---

## 4. Self-Correction

**What:** Detect errors in output and automatically fix them.

**When to Use:**
- Output can be validated
- Fixes are automatable
- Cost of retry < cost of error

**Pattern:**
```
Generate
    │
    ▼
┌─────────────┐
│  VALIDATE   │
└──────┬──────┘
       │
   ┌───┴───┐
   │       │
 Pass    Fail
   │       │
   ▼       ▼
Output  ┌─────────┐
        │ CORRECT │
        │  error  │
        └────┬────┘
             │
             └───► Retry validation
```

**PRP Application:**
```markdown
## Code Self-Correction

### Generate Code
```python
# Initial attempt at auth_service.py
class AuthService:
    def login(self, email, password):
        user = self.db.query(User).filter_by(email=email).first()
        return create_token(user)
```

### Validate
```bash
ruff check src/services/auth_service.py
mypy src/services/auth_service.py
pytest tests/unit/test_auth_service.py -v
```

### Error Detection
```
mypy: error: "db" has no attribute "query"
pytest: FAILED - test_login_invalid_password
```

### Self-Correction
```yaml
correction_loop:
  - error: "db has no attribute query"
    diagnosis: "Wrong SQLAlchemy syntax for async"
    fix: "Use async_session.execute(select(...))"

  - error: "test_login_invalid_password failed"
    diagnosis: "Missing password validation"
    fix: "Add password check before token creation"
```

### Retry
Apply fixes, re-validate until pass or max retries
```

---

## 5. Retrieval-Augmented Generation (RAG)

**What:** Retrieve relevant context before generating output.

**When to Use:**
- External knowledge needed
- Context is large/dynamic
- Accuracy depends on grounding

**Pattern:**
```
Query
  │
  ▼
┌─────────────┐
│  RETRIEVE   │
│  (search)   │ ← Find relevant docs
└──────┬──────┘
       │
       ▼
  [Doc A, Doc B, Doc C]
       │
       ▼
┌─────────────┐
│  AUGMENT    │
│  (inject)   │ ← Add to context
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  GENERATE   │
│  (output)   │ ← Grounded response
└──────┬──────┘
       │
       ▼
    Result
```

**PRP Application:**
```markdown
## Pattern-Aware PRP Generation (RAG)

### Retrieval Phase
```yaml
retrievers:
  codebase:
    query: "similar implementations"
    method: "semantic search over AST"
    top_k: 3

  documentation:
    query: "library best practices"
    method: "web search + fetch"
    top_k: 2

  memory:
    query: "past PRPs for this project"
    method: "memory lookup"
    top_k: 2
```

### Augmentation
```markdown
## Retrieved Context

### From Codebase
- Pattern: `src/services/user_service.py:20-45`
- Pattern: `src/api/routes/users.py:10-30`

### From Documentation
- FastAPI dependency injection: [key insight]
- SQLAlchemy async patterns: [key insight]

### From Memory
- Previous auth PRP used JWT with RS256
- Gotcha: Rate limiting required for auth endpoints
```

### Generation
Generate PRP grounded in retrieved context
```

---

## Combining Planning Patterns

Complex tasks often combine patterns:

```markdown
## Full PRP Builder Workflow

### Phase 1: Plan-and-Execute
Create high-level plan for PRP creation

### Phase 2: ReAct for Discovery
Dynamic exploration of codebase patterns

### Phase 3: RAG for Context
Retrieve and inject relevant information

### Phase 4: Generate Draft
Create initial PRP

### Phase 5: Reflection
Self-critique and identify issues

### Phase 6: Self-Correction
Fix identified issues

### Phase 7: Final Validation
Confirm all criteria met
```

---

## Planning Pattern Decision Guide

| Situation | Pattern | Reason |
|-----------|---------|--------|
| Known steps, complex execution | Plan-and-Execute | Visibility and control |
| Unknown steps, exploration needed | ReAct | Adaptive discovery |
| Quality critical, time available | Reflection | Self-improvement |
| Validation possible, errors fixable | Self-Correction | Automated recovery |
| External knowledge needed | RAG | Grounded accuracy |

---

## Planning Pattern Checklist

When designing planning workflows:

- [ ] Is planning worth the overhead for this task?
- [ ] Are termination conditions clear?
- [ ] Are iteration limits set?
- [ ] Is rollback strategy defined?
- [ ] Can progress be observed?
- [ ] Are reflection criteria specific?
- [ ] Is retrieval quality validated?
