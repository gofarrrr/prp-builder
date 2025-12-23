# Context Scaling Principles

Nine principles for managing context that scales with task complexity.

---

## Core Truth

> "Context engineering is the holistic curation of all information entering the model's limited attention budget."

More context ≠ better results. The goal is **smallest possible set of high-signal tokens** that maximize desired outcomes.

---

## The Nine Principles

### 1. Context Computed, Not Accumulated

**Principle:** Generate context dynamically rather than accumulating it passively.

**Wrong Approach:**
```python
# Accumulates everything
context = []
for turn in conversation:
    context.append(turn)  # Grows without bound
```

**Right Approach:**
```python
# Computes what's needed
def get_context(current_task):
    return [
        get_system_prompt(),
        summarize_session() if len(session) > 5 else session,
        get_relevant_patterns(current_task),
        current_task
    ]
```

**PRP Application:**
```markdown
## Context Requirements
Compute at runtime:
- Relevant patterns from codebase (glob + grep)
- Current task context only
- Compressed session summary

Do NOT accumulate:
- Full conversation history
- All tool outputs
- Every file read
```

---

### 2. Separate Storage from Presentation

**Principle:** Store data in full fidelity; present only what's needed.

**Storage:** Complete, structured, queryable
**Presentation:** Filtered, compressed, relevant

```yaml
# Storage (full)
session_data:
  turns: 47
  tool_calls: 123
  files_read: 34
  full_history: [...]

# Presentation (computed)
current_context:
  summary: "Working on auth feature. 3 tasks complete."
  active_task: "Add JWT middleware"
  relevant_files: ["src/middleware/", "src/auth/"]
```

**PRP Application:**
```markdown
## Data Management
Store (external):
- Full PRP at `PRPs/feature-prp.md`
- Complete task list at `PRPs/feature-tasks.md`

Present (in context):
- Current task only
- Relevant patterns (file:line references)
- Compressed progress summary
```

---

### 3. Scope by Default

**Principle:** Default to minimum scope; expand only when necessary.

| Scope Level | When to Use | Example |
|-------------|-------------|---------|
| **Minimal** | Single file tasks | Bug fix, small change |
| **Feature** | Related files | New endpoint + tests |
| **System** | Cross-cutting | Auth system overhaul |
| **Full** | Almost never | Major refactoring |

**PRP Application:**
```markdown
## Scope Definition
Default scope: `src/services/auth_service.py` only

Expand if needed:
- IF middleware required → add `src/middleware/`
- IF new model needed → add `src/models/`
- IF API changes → add `src/api/routes/`

Never include unless explicitly required:
- Unrelated services
- Infrastructure configs
- Legacy code
```

---

### 4. Retrieval Over Pinning

**Principle:** Retrieve information when needed rather than pinning it permanently.

**Pinning (costly):** Information loaded at start, persists throughout
**Retrieval (efficient):** Information fetched on demand

```python
# Pinning (avoid)
context = load_all_patterns()  # Huge upfront cost

# Retrieval (prefer)
def get_pattern(pattern_type):
    return search_codebase(f"PATTERN:{pattern_type}")
```

**PRP Application:**
```markdown
## Context Loading Strategy
Always loaded (pinned):
- Current task description
- System prompt
- Active file content

Retrieved on demand:
- Similar implementations (via glob/grep)
- External documentation (via web fetch)
- Historical patterns (via memory lookup)
```

---

### 5. Schema-Driven Summarization

**Principle:** Summarize using consistent schemas, not ad-hoc compression.

**Schema for Task Summary:**
```yaml
task_summary_schema:
  task_id: string
  action: "CREATE" | "MODIFY" | "DELETE"
  target: file_path
  outcome: "SUCCESS" | "FAILED" | "PARTIAL"
  key_insight: string (max 50 words)
  next_step: string (optional)
```

**Example:**
```yaml
# Instead of verbose output
task_summary:
  task_id: "T2"
  action: "CREATE"
  target: "src/services/auth_service.py"
  outcome: "SUCCESS"
  key_insight: "Used BaseService pattern, added JWT validation"
  next_step: "Add integration tests"
```

**PRP Application:**
```markdown
## Summarization Schema
After each task, compress to:
- Task: [ID and action]
- Result: [pass/fail + key metric]
- Learned: [one insight]
- Next: [immediate next step]
```

---

### 6. Offload to Tools

**Principle:** Use tools to extend memory rather than context.

**Context-based (limited):**
```markdown
I have read all 50 files in src/ and here they are...
[50 files × 200 lines = 10,000 lines in context]
```

**Tool-based (scalable):**
```markdown
Available tools:
- glob: Find files by pattern
- grep: Search content
- read: Load specific file

Current context: [just the current task]
```

**What to Offload:**
- File search → Glob tool
- Content search → Grep tool
- Full file reads → Read tool with line ranges
- Calculations → Code execution
- Web lookups → Fetch tools

**PRP Application:**
```markdown
## Tool Delegation
Instead of loading context:
- Pattern discovery → `glob "src/**/*service*.py"`
- Find similar code → `grep "class.*Service"`
- Get specifics → `read "src/services/user.py:20-45"`

Keep in context:
- Task description
- Success criteria
- Current file content only
```

---

### 7. Isolate with Sub-Agents

**Principle:** Spawn sub-agents to isolate context-heavy operations.

```
Main Agent (focused context)
    │
    ├─► Sub-Agent: Codebase Analysis
    │   (heavy context, returns summary)
    │
    ├─► Sub-Agent: Documentation Research
    │   (external fetching, returns insights)
    │
    └─► Main continues with summaries only
```

**When to Spawn:**
- Task requires loading many files
- Research phase needs broad search
- Parallel exploration possible
- Context would exceed 50% of window

**PRP Application:**
```markdown
## Sub-Agent Strategy
Main agent handles:
- User interaction
- PRP generation
- Final synthesis

Spawn sub-agents for:
- Codebase pattern discovery (returns: patterns, gotchas, file tree)
- Library research (returns: constraints, examples, versions)
```

---

### 8. Cache Stability

**Principle:** Cache stable context; recompute volatile context.

| Context Type | Stability | Strategy |
|--------------|-----------|----------|
| System prompt | Very stable | Cache always |
| Tool definitions | Stable | Cache per session |
| Project patterns | Mostly stable | Cache with TTL |
| Session state | Volatile | Recompute often |
| Tool outputs | Very volatile | Summarize immediately |

**Implementation:**
```python
context_cache = {
    "system_prompt": {"value": "...", "ttl": "forever"},
    "project_patterns": {"value": "...", "ttl": "1 hour"},
    "tool_definitions": {"value": "...", "ttl": "session"}
}

def get_context():
    stable = load_from_cache(["system_prompt", "tool_definitions"])
    volatile = compute_current_state()
    return stable + volatile
```

**PRP Application:**
```markdown
## Caching Strategy
Cache (stable):
- Project conventions from `~/.prp-cache/`
- Template structures
- Known gotchas

Recompute (volatile):
- Current codebase state
- Session progress
- Active task context
```

---

### 9. Observability First

**Principle:** Make context usage visible and measurable.

**Track:**
- Token count per context component
- Cache hit/miss rates
- Compression ratios
- Sub-agent spawning frequency

**Example Metrics:**
```yaml
context_metrics:
  total_tokens: 45000
  breakdown:
    system_prompt: 500 (1%)
    tool_definitions: 2000 (4%)
    session_history: 8000 (18%)
    tool_outputs: 28000 (62%)  # Target for compression
    current_task: 6500 (14%)

  compression_applied: true
  sub_agents_spawned: 2
```

**PRP Application:**
```markdown
## Context Monitoring
Track per PRP session:
- Starting context size
- Peak context usage
- Compression events
- Sub-agent delegations

Flag if:
- Tool outputs >50% of context
- Session history >30% of context
- Compression triggered >3 times
```

---

## Principles Summary

| # | Principle | Key Action |
|---|-----------|------------|
| 1 | Computed not accumulated | Generate context dynamically |
| 2 | Storage ≠ Presentation | Store full, present filtered |
| 3 | Scope by default | Start minimal, expand when needed |
| 4 | Retrieval over pinning | Fetch on demand, don't preload |
| 5 | Schema-driven summaries | Consistent compression format |
| 6 | Offload to tools | Extend with tools, not context |
| 7 | Isolate with sub-agents | Spawn for heavy operations |
| 8 | Cache stability | Cache stable, recompute volatile |
| 9 | Observability first | Measure and monitor usage |

---

## Quick Application Checklist

Before finalizing a PRP:

- [ ] Is context computed or accumulated?
- [ ] Is storage separated from presentation?
- [ ] Is scope minimal by default?
- [ ] Are heavy loads retrieved, not pinned?
- [ ] Are summaries schema-driven?
- [ ] Are appropriate tasks offloaded to tools?
- [ ] Should sub-agents handle any sections?
- [ ] Is caching applied to stable elements?
- [ ] Can context usage be observed?
