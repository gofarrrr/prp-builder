# Context Engineering for PRPs

> "Context engineering is the holistic curation of all information entering the model's limited attention budget."

## Core Principle

Find the **smallest possible set of high-signal tokens** that maximize the likelihood of desired outcomes. More context ≠ better results.

---

## The Five Context Components

### 1. System Prompts
- Establish agent identity and behavioral guidelines
- Balance specificity with flexibility
- Place at beginning of context (attention-favored position)

### 2. Tool Definitions
- Names, descriptions, parameters, formats
- Quality descriptions include usage context and examples
- Rule: "If a human engineer cannot definitively say which tool to use, an agent cannot do better"

### 3. Retrieved Documents
- Domain-specific knowledge loaded dynamically
- Use lightweight references over full documents
- Pre-load essentials; retrieve the rest on-demand

### 4. Message History
- Conversation record as scratchpad memory
- Tracks progress and maintains task state
- Older turns are candidates for compression

### 5. Tool Outputs
- Results of agent actions
- **Consume up to 83.9% of total context**
- Primary target for compression/masking

---

## Context Degradation Patterns

### Lost-in-Middle
Information buried in context receives 10-40% lower recall than content at boundaries.

**Mitigation:** Place critical info at start/end of sections.

### Context Poisoning
Errors from tool outputs compound through repeated reference.

**Mitigation:** Validate tool outputs; don't propagate errors.

### Context Distraction
Irrelevant information competes for attention. Even one irrelevant document reduces performance.

**Mitigation:** Aggressive filtering; include only what's necessary.

### Context Confusion
Models incorporate incompatible information when multiple task types share context.

**Mitigation:** Single responsibility per PRP; clear task boundaries.

### Context Clash
Contradictory information from multiple sources derails reasoning.

**Mitigation:** Explicit precedence rules; version-specific references.

---

## Optimization Strategies

### Progressive Disclosure
Load information only when needed:
```
Startup → Skill name + description only
Phase 2 → Relevant template sections
Phase 3 → Discovered codebase patterns
Phase 4 → Full generation template
```

### Compaction Triggers
Activate at 70-80% context utilization:
- Summarize older conversation turns
- Replace verbose tool outputs with references
- Compress retrieved documents to key points
- **Never compress:** System prompts, current task instructions

### Observation Masking
Tool outputs often consume 80%+ of tokens:
- Replace verbose outputs with compact references
- Keep accessibility for potential retrieval
- Prioritize recent outputs over older ones

### Context Partitioning
Distribute work across isolated sub-agents:
- Each operates in clean, focused context
- Coordinator handles synthesis
- Prevents single massive context accumulation

---

## PRP-Specific Application

### In Task Sections
```markdown
## Task 1: CREATE `api/routes/auth.py`
- PATTERN: `api/routes/users.py:12-45`  # Reference, don't copy
- IMPORTS: FastAPI, Depends, HTTPException  # Minimal, specific
- VALIDATE: `pytest tests/test_auth.py -v`  # Immediately after task
```

### In Context Sections
```yaml
# Good: Specific, anchored references
documentation:
  - url: "https://docs.lib.com/auth#jwt-setup"
    insight: "Use RS256 for production"
  - file: "src/config/settings.py:23-30"
    insight: "JWT_SECRET from env"

# Bad: Broad, unanchored references
documentation:
  - "FastAPI docs"
  - "Our authentication module"
```

### In Validation
```markdown
# Good: Executable, immediate
VALIDATE: `ruff check src/auth/ && pytest tests/auth/ -v`

# Bad: Vague, delayed
"Make sure to test the authentication"
```

---

## Token Budget Guidelines

| PRP Type | Target Size | Context Budget |
|----------|-------------|----------------|
| Task | 200-500 tokens | Single file scope |
| Story | 500-1500 tokens | Feature scope |
| Base | 1500-4000 tokens | System scope |
| Planning | 2000-5000 tokens | Research + spec |

**Rule of thumb:** If an agent needs to scroll extensively, the PRP is too long. Compress or split.

---

## Quality Checklist

Before finalizing any PRP:

- [ ] Critical constraints at section boundaries?
- [ ] References specific (file:line) not vague?
- [ ] Tool outputs compressed to essentials?
- [ ] Single responsibility per task?
- [ ] No contradictory instructions?
- [ ] Validation immediately after each task?
- [ ] Could be implemented without additional research?
