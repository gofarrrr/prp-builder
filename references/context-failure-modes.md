# Context Failure Modes

Nine ways context management fails and how to prevent each.

---

## Overview

Context failures cause agents to produce incorrect, inconsistent, or degraded outputs. Each failure mode has specific detection signals and targeted fixes.

---

## 1. Context Poisoning

**What:** Errors in tool outputs propagate through context, compounding over time.

**Symptoms:**
- Agent references incorrect data from earlier
- Errors cascade through multiple tasks
- "Hallucinated" information that came from a failed tool call

**Example:**
```
Tool output (corrupted): {"user_count": -1}
Agent uses this: "Based on the -1 users in the system..."
Propagates: All subsequent analysis assumes negative users
```

**Fix:**
```markdown
## Validation Protocol
After each tool call:
1. Validate output structure matches expected schema
2. Validate values are within reasonable bounds
3. IF invalid: discard output, retry with different approach
4. NEVER propagate unvalidated tool outputs
```

**PRP Application:**
```markdown
### Task Validation
- VALIDATE: Check output structure before using
- IF_INVALID: Log error, do not proceed
- SCHEMA: `{"status": "ok|error", "data": {...}}`
```

---

## 2. Context Distraction

**What:** Irrelevant information competes for attention, degrading focus.

**Symptoms:**
- Agent mentions unrelated information
- Responses include tangential details
- Performance degrades as context grows

**Research Finding:** Even ONE irrelevant document in context measurably reduces performance.

**Example:**
```markdown
# Bad: Includes irrelevant context
## Context
- User authentication (relevant)
- Payment processing (irrelevant)
- Notification system (irrelevant)
- Email templates (irrelevant)
```

**Fix:**
```markdown
## Context
- User authentication ONLY
  - File: `src/auth/handler.py`
  - Pattern: `src/auth/oauth.py:20-45`

All other systems explicitly excluded from scope.
```

**PRP Application:**
```markdown
## Scope Boundaries
IN SCOPE:
- `src/auth/*` - target files

OUT OF SCOPE (do not load):
- `src/payments/*`
- `src/notifications/*`
- `src/email/*`
```

---

## 3. Context Confusion

**What:** Multiple task types share context, causing mixed behaviors.

**Symptoms:**
- Response style switches mid-task
- Agent applies wrong patterns to task
- Outputs combine formats from different contexts

**Example:**
```markdown
# Confusing: Mixed task types in one PRP
## Tasks
1. Write unit tests (testing mindset)
2. Update documentation (writing mindset)
3. Refactor for performance (optimization mindset)
4. Add user-facing messages (UX mindset)
```

**Fix:**
```markdown
# Clear: Single responsibility per PRP
## PRP 1: Testing
Focus: Unit tests only

## PRP 2: Documentation
Focus: Docs only

## PRP 3: Performance
Focus: Optimization only
```

**PRP Application:**
```markdown
## Single Responsibility
This PRP covers: Authentication service implementation ONLY

Separate PRPs needed for:
- Testing (create after implementation)
- Documentation (create after testing)
- Performance optimization (create if needed)
```

---

## 4. Context Clash

**What:** Contradictory information from multiple sources causes confusion.

**Symptoms:**
- Agent alternates between contradictory approaches
- Outputs contain logical inconsistencies
- Agent explicitly notes confusion about requirements

**Example:**
```markdown
## Requirements (contradictory)
- Documentation says: "Use synchronous calls"
- Code pattern shows: async/await everywhere
- User says: "Follow existing patterns"
```

**Fix:**
```markdown
## Requirements (clear precedence)
1. User instructions (highest priority)
2. Existing code patterns
3. External documentation (lowest priority)

RESOLVED: Use async/await (matches code patterns + user preference)
```

**PRP Application:**
```markdown
## Precedence Rules
IF conflict between sources:
1. Explicit user instruction wins
2. Existing codebase patterns win
3. External documentation is advisory only

Current decision: [state resolved approach]
```

---

## 5. Lost in Middle

**What:** Information in middle of context receives less attention than edges.

**Symptoms:**
- Agent misses requirements stated in middle of document
- Follows instructions from beginning/end, ignores middle
- Key constraints forgotten during execution

**Research:** Middle content receives 10-40% lower recall than boundary content.

**Example:**
```markdown
## Requirements
1. Create user model ← Agent remembers (beginning)
2. Add email validation
3. Implement password hashing
4. Set up rate limiting  ← Agent forgets (middle)
5. Add audit logging
6. Write comprehensive tests ← Agent remembers (end)
```

**Fix:**
```markdown
## Critical Requirements (repeated at boundaries)
> Rate limiting is REQUIRED for this feature

## Requirements
1. Create user model
2. Add email validation
3. Implement password hashing
4. **Set up rate limiting** ← Highlighted
5. Add audit logging
6. Write comprehensive tests

## Validation (reinforces critical items)
- [ ] Rate limiting implemented and tested
```

**PRP Application:**
```markdown
## Critical Constraints
[State most important constraints at TOP]

## Implementation Tasks
[Detailed tasks here]

## Reminder: Critical Constraints
[Repeat critical constraints at BOTTOM]
```

---

## 6. Attention Saturation

**What:** Context window fills beyond useful capacity.

**Symptoms:**
- Responses become less coherent
- Agent misses obvious patterns
- Performance noticeably degrades
- Earlier context seems forgotten

**Thresholds:**
| Usage | Status | Action |
|-------|--------|--------|
| <50% | Healthy | Continue |
| 50-70% | Caution | Monitor |
| 70-85% | Warning | Compress |
| >85% | Critical | Delegate or split |

**Fix:**
```markdown
## Context Budget
Current usage: [estimated tokens]
Threshold: 70% of window

IF threshold exceeded:
1. Summarize older conversation turns
2. Compress tool outputs to key findings
3. Consider spawning sub-agent for remaining work
```

**PRP Application:**
```markdown
## Token Budget
Target PRP size: <2000 tokens
Task sections: ~300 tokens each
Context references: File:line only, not full content

IF PRP exceeds budget:
- Split into multiple PRPs
- Move details to external files
- Use reference patterns over inline code
```

---

## 7. Recency Bias

**What:** Recent context overly influences decisions, ignoring earlier requirements.

**Symptoms:**
- Latest user message overrides earlier constraints
- Agent "forgets" requirements from session start
- Solutions align with recent context but miss original goals

**Example:**
```
Turn 1: "Build secure auth system with rate limiting"
Turn 5: "Make the login form pretty"
Turn 10: Agent focuses entirely on UI, forgets security requirements
```

**Fix:**
```markdown
## Requirements Anchoring
ORIGINAL GOAL (turn 1): Secure auth with rate limiting
CURRENT TASK (turn 10): UI improvements

CONSTRAINT: UI work must not compromise security requirements.
VALIDATION: Security tests must still pass after UI changes.
```

**PRP Application:**
```markdown
## Goal Anchoring
Primary goal: [from original request]
Current subtask: [specific current work]

Success requires BOTH:
- [ ] Primary goal criteria met
- [ ] Current subtask completed
```

---

## 8. Authority Confusion

**What:** Agent unclear which source has authority when instructions conflict.

**Symptoms:**
- Agent asks unnecessary clarification questions
- Switches between conflicting approaches
- Cites different sources for contradictory actions

**Sources (typical authority order):**
1. Explicit user instructions
2. System prompt
3. Tool definitions
4. Retrieved documents
5. Training knowledge

**Example:**
```
System prompt: "Always use TypeScript"
User: "Write this in Python"
Retrieved doc: "JavaScript recommended"
→ Agent confused about which language
```

**Fix:**
```markdown
## Authority Hierarchy
1. User instructions (this conversation)
2. System/skill prompts
3. Project conventions (from codebase)
4. External documentation

CURRENT AUTHORITY: User requested Python
(Overrides system default of TypeScript)
```

**PRP Application:**
```markdown
## Decision Authority
For this PRP:
- Language: Python (user specified)
- Framework: FastAPI (project convention)
- Style: Black + Ruff (project linter config)

IF conflict: Ask user before proceeding
```

---

## 9. Format Lock-in

**What:** Agent rigidly follows format even when inappropriate.

**Symptoms:**
- Responses use same structure regardless of question type
- Forced formatting where none is needed
- Awkward template application

**Example:**
```markdown
User: "What's 2+2?"

Agent response (format locked):
## Problem Analysis
The user is asking for mathematical computation.

## Approach
We will use arithmetic addition.

## Solution
2 + 2 = 4

## Validation
The answer is verified.
```

**Fix:**
Allow format flexibility based on task:
```markdown
## Response Formatting
| Task Type | Format |
|-----------|--------|
| Simple question | Direct answer |
| Implementation | Structured PRP |
| Debugging | Diagnostic flow |
| Research | Summary with sources |
```

**PRP Application:**
```markdown
## Adaptive Formatting
This PRP uses [template type] because:
- Scope: [small/medium/large]
- Complexity: [simple/moderate/complex]

Skip sections not relevant to this task.
```

---

## Failure Mode Detection Checklist

Use during PRP review:

| # | Failure Mode | Detection Question |
|---|--------------|-------------------|
| 1 | Poisoning | Are all tool outputs validated before use? |
| 2 | Distraction | Is irrelevant context excluded? |
| 3 | Confusion | Is there single responsibility per PRP? |
| 4 | Clash | Are conflicts resolved with clear precedence? |
| 5 | Lost in Middle | Are critical items at section boundaries? |
| 6 | Saturation | Is token budget monitored? |
| 7 | Recency Bias | Is original goal anchored throughout? |
| 8 | Authority | Is source hierarchy explicit? |
| 9 | Format Lock-in | Is format appropriate to task type? |

---

## Quick Fixes Summary

| Failure | Quick Fix |
|---------|-----------|
| Poisoning | Validate tool outputs before use |
| Distraction | Explicit scope boundaries |
| Confusion | Single responsibility per PRP |
| Clash | State precedence rules |
| Lost in Middle | Repeat critical items at boundaries |
| Saturation | Compress at 70% usage |
| Recency Bias | Anchor original goal throughout |
| Authority | Explicit authority hierarchy |
| Format Lock-in | Match format to task type |
