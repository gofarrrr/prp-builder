# Sequential Design Patterns

Patterns for linear, step-by-step agent workflows.

---

## Overview

Sequential patterns process work in ordered steps. Each step completes before the next begins.

```
Input → Step 1 → Step 2 → Step 3 → Output
```

---

## 1. Prompt Chaining

**What:** Break complex tasks into sequential LLM calls, each transforming the output of the previous.

**When to Use:**
- Task has clear stages
- Each stage produces distinct output
- Later stages depend on earlier results

**Pattern:**
```
User Request
    │
    ▼
┌─────────────┐
│  Stage 1:   │
│  Parse      │ → Structured requirements
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Stage 2:   │
│  Research   │ → Context & patterns
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Stage 3:   │
│  Generate   │ → PRP draft
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Stage 4:   │
│  Validate   │ → Final PRP
└─────────────┘
```

**PRP Application:**
```markdown
## Workflow: PRP Creation Chain

### Chain Step 1: Requirements Extraction
- INPUT: User description
- OUTPUT: Structured requirements JSON
- VALIDATE: All required fields present

### Chain Step 2: Codebase Analysis
- INPUT: Requirements + project path
- OUTPUT: Patterns, gotchas, file structure
- VALIDATE: At least one pattern found

### Chain Step 3: Template Population
- INPUT: Requirements + analysis
- OUTPUT: Draft PRP
- VALIDATE: All sections filled

### Chain Step 4: Quality Check
- INPUT: Draft PRP
- OUTPUT: Final PRP with fixes
- VALIDATE: Context completeness check passes
```

**Gate Between Stages:**
```markdown
## Stage Transition
PROCEED to next stage IF:
- Current stage output is valid
- Required fields are present
- No blocking errors

RETRY current stage IF:
- Output invalid
- Missing required data
- Recoverable error

FAIL workflow IF:
- 3 retries exhausted
- Unrecoverable error
- User cancellation
```

---

## 2. Routing

**What:** Classify input and direct to appropriate handler.

**When to Use:**
- Multiple task types possible
- Different handling required per type
- Classification is reliable

### LLM-Based Routing

```
User Input
    │
    ▼
┌─────────────┐
│   Router    │
│   (LLM)     │
└──────┬──────┘
       │
   ┌───┴───┐───────┐
   │       │       │
   ▼       ▼       ▼
┌─────┐ ┌─────┐ ┌─────┐
│Task │ │Story│ │Base │
│ PRP │ │ PRP │ │ PRP │
└─────┘ └─────┘ └─────┘
```

**PRP Application:**
```markdown
## Routing Logic

### Classification Prompt
Analyze the user request and classify:
- TASK: Single file, <1 hour, atomic change
- STORY: Multi-file, 1-3 days, scoped feature
- BASE: System-wide, >3 days, major feature
- PLANNING: Research needed, unclear scope

### Route Handlers
| Classification | Template | Next Step |
|----------------|----------|-----------|
| TASK | `templates/task.md` | Direct generation |
| STORY | `templates/story.md` | Gather sprint context |
| BASE | `templates/base.md` | Full discovery phase |
| PLANNING | `templates/planning.md` | Research phase first |
```

### Rule-Based Routing

```markdown
## Rule-Based Router

IF user mentions "bug" OR "fix" OR "typo":
  → ROUTE to Task PRP

IF user mentions "feature" AND scope < 3 files:
  → ROUTE to Story PRP

IF user mentions "system" OR "architecture":
  → ROUTE to Base PRP

IF user says "not sure" OR "explore":
  → ROUTE to Planning PRP

DEFAULT:
  → ASK user for clarification
```

---

## 3. Fallback Pattern

**What:** Try primary approach; if fails, cascade to alternatives.

**When to Use:**
- Primary approach might fail
- Alternative approaches exist
- Graceful degradation preferred over failure

**Pattern:**
```
Input
  │
  ▼
┌─────────────────┐
│  Primary Path   │──── Success ───► Output
└────────┬────────┘
         │ Fail
         ▼
┌─────────────────┐
│ Fallback Path 1 │──── Success ───► Output
└────────┬────────┘
         │ Fail
         ▼
┌─────────────────┐
│ Fallback Path 2 │──── Success ───► Output
└────────┬────────┘
         │ Fail
         ▼
    Human Escalation
```

**PRP Application:**
```markdown
## Codebase Pattern Discovery

### Primary: Glob Search
```bash
glob "src/**/*service*.py"
```
IF found → Extract patterns
IF empty → Fallback 1

### Fallback 1: Grep Search
```bash
grep "class.*Service" --type py
```
IF found → Extract patterns
IF empty → Fallback 2

### Fallback 2: Ask User
"I couldn't find existing service patterns.
Please provide a reference file or describe
the pattern you want to follow."

### Fallback 3: Use Defaults
IF user doesn't provide:
→ Use standard Python service patterns
→ Document assumption in PRP
```

---

## 4. Gate Pattern

**What:** Quality checkpoint that must pass before proceeding.

**When to Use:**
- Critical validation points
- Expensive operations ahead
- Error recovery is costly

**Pattern:**
```
Step N
  │
  ▼
┌─────────────┐
│    GATE     │
│  Condition  │
└──────┬──────┘
       │
   ┌───┴───┐
   │       │
 PASS    FAIL
   │       │
   ▼       ▼
Step N+1  Remediation
             │
             └──► Retry Gate
```

**PRP Application:**
```markdown
## Quality Gates

### Gate 1: Context Completeness
BEFORE generating PRP:
- [ ] All file paths are specific (file:line)
- [ ] Acceptance criteria are executable
- [ ] Dependencies are explicit
- [ ] Patterns are referenced, not described

IF ANY unchecked → Gather missing context
IF ALL checked → Proceed to generation

### Gate 2: Validation Pass
BEFORE marking task complete:
```bash
ruff check src/ && pytest tests/ -v
```
IF exit code 0 → Mark complete
IF exit code != 0 → Fix issues, retry

### Gate 3: Human Approval
BEFORE deployment tasks:
- Present changes summary
- Wait for explicit "proceed" or "approved"
- Log approval for audit
```

---

## 5. Human-in-the-Loop

**What:** Pause workflow for human input at critical points.

**When to Use:**
- Irreversible actions
- High-stakes decisions
- Ambiguous requirements
- Learning opportunities

**Pattern:**
```
Automated Steps
      │
      ▼
┌─────────────┐
│   PAUSE     │
│  for Human  │◄─── Present options
└──────┬──────┘     Wait for input
       │
   Human Input
       │
       ▼
Continue Automated Steps
```

**PRP Application:**
```markdown
## Human Checkpoints

### Checkpoint 1: Template Selection
PAUSE and present:
"Based on your description, I recommend [TEMPLATE].
Options:
1. Task PRP (simple, <1 hour)
2. Story PRP (medium, 1-3 days)
3. Base PRP (complex, >3 days)

Your choice?"

### Checkpoint 2: Scope Confirmation
PAUSE and present:
"I've identified these files in scope:
- `src/services/auth.py` (create)
- `src/models/user.py` (modify)
- `tests/test_auth.py` (create)

Confirm scope or adjust?"

### Checkpoint 3: Destructive Operations
PAUSE before:
- Database migrations
- File deletions
- Production deployments
- Irreversible changes

"This will [ACTION]. Type 'proceed' to continue."
```

---

## Combining Sequential Patterns

Real workflows often combine patterns:

```markdown
## PRP Builder Workflow

1. INPUT PROCESSING (Prompt Chaining)
   Parse → Validate → Structure

2. TEMPLATE SELECTION (Routing)
   Classify → Route to handler

3. CONTEXT GATHERING (Fallback)
   Primary search → Fallback search → Ask user

4. QUALITY CHECK (Gate)
   Must pass completeness check

5. USER REVIEW (Human-in-the-Loop)
   Present draft → Get approval

6. OUTPUT GENERATION (Prompt Chaining)
   Generate → Validate → Finalize
```

---

## Sequential Pattern Checklist

When designing sequential workflows:

- [ ] Each step has clear input/output?
- [ ] Steps are properly ordered?
- [ ] Gates exist at critical points?
- [ ] Fallbacks handle failures gracefully?
- [ ] Human checkpoints at high-stakes decisions?
- [ ] Retry logic is bounded (max 3)?
- [ ] Escalation path is defined?
