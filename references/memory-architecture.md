# Memory Architecture

Four-layer memory model for managing agent state across contexts.

---

## Overview

Agent memory operates in four distinct layers, each with different scope, lifetime, and purpose:

```
┌─────────────────────────────────────────────────────────┐
│                   MEMORY ARCHITECTURE                    │
├─────────────────────────────────────────────────────────┤
│  Layer 4: ARTIFACTS     │ Files, code, documents        │
│  (Permanent)            │ External storage              │
├─────────────────────────────────────────────────────────┤
│  Layer 3: MEMORY        │ Cross-session persistence     │
│  (Long-term)            │ User preferences, patterns    │
├─────────────────────────────────────────────────────────┤
│  Layer 2: SESSIONS      │ Conversation history          │
│  (Medium-term)          │ Multi-turn context            │
├─────────────────────────────────────────────────────────┤
│  Layer 1: WORKING       │ Current turn processing       │
│  (Short-term)           │ Active reasoning state        │
└─────────────────────────────────────────────────────────┘
```

---

## Layer 1: Working Context

**Scope:** Current turn only
**Lifetime:** Milliseconds to seconds
**Purpose:** Active reasoning and decision-making

### What Lives Here
- Current user message
- Active tool outputs
- Immediate reasoning state
- Selected context from other layers

### Management Strategy
```markdown
## Working Context Budget
- System prompt: ~500 tokens (fixed)
- Current message: variable
- Tool definitions: ~200 tokens per tool
- Active tool outputs: compress aggressively
- Retrieved context: scope to current task only

Target: <30% of context window for working context
```

### PRP Application
```markdown
## Current Task Context
- User request: [exact user message]
- Active file: `src/services/auth.py`
- Recent output: [compressed validation result]
```

---

## Layer 2: Sessions

**Scope:** Current conversation
**Lifetime:** Minutes to hours
**Purpose:** Multi-turn coherence and task tracking

### What Lives Here
- Conversation history
- Completed task summaries
- Discovered patterns this session
- User clarifications and decisions

### Management Strategy
```markdown
## Session Management
- Compress older turns after 5 exchanges
- Keep: user decisions, discovered patterns, task outcomes
- Drop: verbose tool outputs, superseded attempts
- Summary format: "[Task] → [Outcome] → [Learned]"
```

### Compression Triggers
| Condition | Action |
|-----------|--------|
| >10 conversation turns | Summarize turns 1-7 |
| Tool output >1000 tokens | Replace with reference |
| >70% context used | Aggressive compression |
| Task completed | Archive to summary |

### PRP Application
```markdown
## Session State
Completed:
- [x] Created `auth_service.py` following user pattern
- [x] Tests pass (5/5)

Discovered:
- Project uses `BaseService` pattern
- Tests go in `tests/unit/` not `tests/`

Pending:
- [ ] Add middleware integration
```

---

## Layer 3: Memory

**Scope:** Cross-session persistence
**Lifetime:** Days to weeks
**Purpose:** Long-term learning and preferences

### What Lives Here
- User preferences and patterns
- Project-specific conventions
- Frequently used file locations
- Historical decisions and rationale

### Storage Patterns
```yaml
# Example: ~/.prp-cache/project-memory.yaml
project: my-api
patterns:
  service_base: "src/services/base.py"
  test_convention: "tests/unit/{module}/test_{file}.py"
  response_format: "src/schemas/base.py:ResponseModel"

preferences:
  framework: FastAPI
  orm: SQLAlchemy 2.0
  testing: pytest + httpx

gotchas:
  - "Always use async session"
  - "Pydantic v2: model_validate not parse_obj"
```

### When to Write
- User explicitly states preference
- Pattern used successfully 3+ times
- Gotcha encountered and resolved

### When to Read
- Starting new PRP for known project
- User references "our pattern" or "like before"
- Template selection phase

### PRP Application
```markdown
## Project Memory
Loaded from: `~/.prp-cache/my-api.yaml`

Patterns:
- Services: Follow `src/services/base.py`
- Tests: `tests/unit/{module}/test_{file}.py`

Preferences:
- Async-first architecture
- Pydantic v2 models
```

---

## Layer 4: Artifacts

**Scope:** External files and documents
**Lifetime:** Permanent (until deleted)
**Purpose:** Durable outputs and references

### What Lives Here
- Generated PRPs
- Code files
- Configuration files
- Documentation
- Test files

### Storage Strategy
```markdown
## Artifact Organization
PRPs/
├── feature-name-prp.md      # Completed PRPs
├── feature-name-tasks.md    # Generated task lists
└── archive/                 # Historical PRPs

src/
└── [generated code files]

docs/
└── [generated documentation]
```

### Artifact References
Instead of loading full artifacts into context:
```markdown
# Good: Reference with specific location
See implementation pattern at `src/services/auth.py:45-67`

# Bad: Load entire file into context
[full file content here]
```

### PRP Application
```markdown
## Artifacts
Output: `PRPs/user-auth-prp.md`
Related:
  - `PRPs/api-routes-prp.md` (similar scope)
  - `src/services/user_service.py` (pattern reference)
```

---

## Layer Interactions

### Read Flow (Bottom-Up)
```
Artifacts → Memory → Session → Working
   │          │         │         │
   └──────────┴─────────┴─────────┘
         All feed into active context
```

### Write Flow (Top-Down)
```
Working → Session → Memory → Artifacts
   │         │         │         │
   │         │         │         └─ Permanent files
   │         │         └─ Patterns used 3+ times
   │         └─ Completed task summaries
   └─ Immediate reasoning state
```

---

## Memory Operations

### 1. Context Loading
```python
# Pseudocode for loading context
def load_context(task):
    context = []

    # Layer 4: Load relevant artifacts (references only)
    context += get_artifact_references(task.related_files)

    # Layer 3: Load persistent memory
    context += load_project_memory(task.project)

    # Layer 2: Load session state
    context += get_session_summary()

    # Layer 1: Add current task
    context += [task.description]

    return compress_if_needed(context)
```

### 2. State Updates
```python
# When to update each layer
def update_memory(event):
    if event.type == "tool_output":
        update_working_context(event.data)

    if event.type == "task_complete":
        update_session_summary(event.task)

    if event.type == "pattern_confirmed":
        persist_to_memory(event.pattern)

    if event.type == "file_written":
        register_artifact(event.file_path)
```

### 3. Compression
```python
# Compression strategies by layer
compression_strategies = {
    "working": "Replace verbose outputs with summaries",
    "session": "Summarize older turns, keep decisions",
    "memory": "Merge similar patterns, prune unused",
    "artifacts": "Never compress - external storage"
}
```

---

## PRP Memory Checklist

When designing PRPs, consider each memory layer:

### Layer 1: Working Context
- [ ] What context is needed for current task?
- [ ] What tool outputs are expected?
- [ ] How will outputs be compressed?

### Layer 2: Session
- [ ] What state persists across tasks?
- [ ] When should tasks be summarized?
- [ ] What decisions need tracking?

### Layer 3: Memory
- [ ] What patterns should be remembered?
- [ ] What preferences apply?
- [ ] What gotchas were discovered?

### Layer 4: Artifacts
- [ ] Where are outputs saved?
- [ ] What files are referenced?
- [ ] How are artifacts organized?

---

## Common Memory Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Loading full files | Context overflow | Use references with line numbers |
| No session compression | Runs out of context | Summarize after each task |
| Forgetting patterns | Repeats mistakes | Persist to Layer 3 after 3 uses |
| Inline artifacts | Bloated context | Save to files, reference by path |
| No state tracking | Loses progress | Explicit session state section |
