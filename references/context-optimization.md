# Context Optimization

Token budgets, compression strategies, and optimization patterns for PRPs.

---

## Token Budget Guidelines

### By PRP Type

| PRP Type | Target Size | Context Budget | Use Case |
|----------|-------------|----------------|----------|
| **Task** | 200-500 tokens | Single file scope | Bug fixes, small changes |
| **Story** | 500-1500 tokens | Feature scope | Sprint tasks, user stories |
| **Base** | 1500-3000 tokens | System scope | Major features |
| **Planning** | 2000-4000 tokens | Research + spec | Unclear requirements |
| **Generative AI** | 2000-4000 tokens | AI system scope | LLM integrations |

### Rule of Thumb
> If an agent needs to scroll extensively, the PRP is too long. Compress or split.

---

## Five Context Components

Understand what consumes context and optimize each:

### 1. System Prompts (~500 tokens)
- **Optimization:** Keep concise, essential only
- **Never compress:** Core behavioral instructions

### 2. Tool Definitions (~200 tokens per tool)
- **Optimization:** Only load tools needed for current phase
- **Progressive loading:** Start with core tools, add specialists

### 3. Retrieved Documents (variable)
- **Optimization:** Use references (file:line) not full content
- **Budget:** <20% of total context

### 4. Message History (growing)
- **Optimization:** Summarize older turns
- **Compress at:** >10 turns or >70% usage

### 5. Tool Outputs (up to 83% of context!)
- **Primary target for compression**
- **Mask immediately after processing**

---

## Compression Strategies

### 1. Progressive Disclosure

Load information only when needed:

```
Phase 1: Startup
├── Skill name + description only
└── No templates loaded

Phase 2: Template Selection
├── Template options (summaries)
└── User selects type

Phase 3: Context Gathering
├── Selected template sections
└── Codebase patterns discovered

Phase 4: Generation
├── Full template
└── All discovered context
└── Validation requirements
```

### 2. Compaction Triggers

Activate compression at 70-80% context utilization:

| Trigger | Action |
|---------|--------|
| >10 conversation turns | Summarize turns 1-7 |
| Tool output >1000 tokens | Replace with summary |
| >70% context used | Aggressive compression |
| Task completed | Archive to summary format |

### 3. Observation Masking

Tool outputs are the biggest consumer. Mask after processing:

```markdown
## Before Masking
Tool output: [2000 tokens of file content]

## After Masking
Tool output reference: "Read `src/auth.py` (312 lines).
Key finding: Uses BaseService pattern at line 45."
```

**Keep:**
- Key findings
- Error messages
- Specific line references

**Remove:**
- Full file contents
- Verbose success messages
- Duplicate information

### 4. Context Partitioning

Distribute heavy work across sub-agents:

```
Main Agent (lean context)
├── User conversation
├── Current PRP draft
└── Compressed summaries

Sub-Agent: Codebase Analyst
├── Heavy file loading
├── Pattern discovery
└── Returns: Summary only

Sub-Agent: Library Researcher
├── External documentation
├── API research
└── Returns: Key findings only
```

---

## Optimization Patterns

### Pattern 1: Reference, Don't Copy

```markdown
# Bad: Copies entire pattern
```python
class UserService:
    def __init__(self, db):
        self.db = db
    # ... 50 lines of code
```

# Good: References pattern
- PATTERN: `src/services/user_service.py:12-60`
- KEY: Uses dependency injection, async methods
```

### Pattern 2: Anchored Documentation

```markdown
# Bad: Broad reference
- See FastAPI documentation

# Good: Anchored reference
documentation:
  - url: "https://fastapi.tiangolo.com/tutorial/dependencies/#dependencies-with-yield"
    insight: "Use yield for session cleanup"
```

### Pattern 3: Structured Summaries

```markdown
# Bad: Verbose summary
The codebase analysis revealed that there are several patterns
in use. The services follow a base class pattern and the API
routes use dependency injection. Testing is done with pytest...

# Good: Structured summary
## Codebase Analysis
- Services: `BaseService` pattern at `src/services/base.py`
- Routes: DI via `Depends()`
- Tests: pytest + fixtures in `conftest.py`
```

### Pattern 4: Scope Minimization

```markdown
# Bad: Kitchen sink scope
## Files to Consider
- All files in src/
- All tests
- All config files
- All documentation

# Good: Minimal scope
## Files in Scope
- `src/services/auth_service.py` (create)
- `src/services/base.py` (reference pattern)
- `tests/unit/services/test_auth.py` (create)
```

---

## PRP-Specific Optimization

### Task Sections
```markdown
## Task 1: CREATE `api/routes/auth.py`
- PATTERN: `api/routes/users.py:12-45`  # Reference, don't copy
- IMPORTS: FastAPI, Depends, HTTPException  # Minimal list
- VALIDATE: `pytest tests/test_auth.py -v`  # Immediate validation
```

### Context Sections
```yaml
# Good: Specific, minimal
documentation:
  - url: "https://docs.example.com/auth#jwt"
    insight: "Use RS256 for production"
  - file: "src/config/settings.py:23-30"
    insight: "JWT_SECRET from env"

# Bad: Broad, wasteful
documentation:
  - "FastAPI docs"
  - "Our authentication guide"
  - "Security best practices"
```

### Validation Sections
```markdown
# Good: Executable, immediate
VALIDATE: `ruff check src/auth/ && pytest tests/auth/ -v`

# Bad: Vague, delayed
"Make sure to test the authentication when you're done"
```

---

## Token Counting Heuristics

Quick estimation without exact counting:

| Content Type | Rough Estimate |
|--------------|----------------|
| 1 word | ~1.3 tokens |
| 1 line of code | ~10-15 tokens |
| 1 paragraph | ~50-100 tokens |
| 1 function (20 lines) | ~200-300 tokens |
| 1 full file (200 lines) | ~2000-3000 tokens |

### PRP Size Estimates

| Section | Typical Tokens |
|---------|----------------|
| Metadata | 50-100 |
| Goal | 100-200 |
| Context | 200-500 |
| Each Task | 150-300 |
| Validation | 100-200 |
| Gotchas | 100-200 |

---

## Optimization Checklist

Before finalizing any PRP:

### Size Check
- [ ] Total PRP within target budget for type?
- [ ] No section exceeds 30% of total?
- [ ] Can be read without scrolling?

### Compression Check
- [ ] All references use file:line format?
- [ ] No full file contents embedded?
- [ ] Tool outputs summarized?
- [ ] Older context compressed?

### Scope Check
- [ ] Only necessary files in scope?
- [ ] Irrelevant systems excluded?
- [ ] Single responsibility maintained?

### Progressive Disclosure
- [ ] Information loaded only when needed?
- [ ] Heavy operations delegated to tools?
- [ ] Sub-agents used for broad searches?

---

## Emergency Compression

When context is critically full (>85%):

1. **Immediate actions:**
   - Summarize all tool outputs to 1-2 sentences each
   - Compress conversation history to key decisions only
   - Remove any duplicate information

2. **Structural actions:**
   - Split remaining work into new PRP
   - Spawn sub-agent for remaining tasks
   - Save current state to artifact file

3. **Prevention:**
   - Set compression triggers earlier (60%)
   - Use smaller scope PRPs
   - Delegate earlier to sub-agents
