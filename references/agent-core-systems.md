# Agent Core Systems

Four fundamental systems that power agentic AI applications.

---

## Overview

Every effective AI agent comprises four interconnected systems:

```
┌─────────────────────────────────────────────────────────┐
│                    AGENT ARCHITECTURE                    │
├──────────────┬──────────────┬──────────────┬────────────┤
│  PERCEPTION  │  REASONING   │    MEMORY    │ EXECUTION  │
│   (Input)    │  (Decision)  │   (State)    │  (Action)  │
└──────────────┴──────────────┴──────────────┴────────────┘
```

---

## 1. Perception System

**Purpose:** Process and interpret all inputs before reasoning.

### Components
- **Input parsing** - Structured extraction from user messages
- **Context retrieval** - Pull relevant information from memory
- **Tool output processing** - Parse and validate action results
- **Multimodal handling** - Images, documents, code, data

### PRP Application
```markdown
## Perception Requirements
- Input format: JSON API requests
- Context sources: `src/services/context_loader.py`
- Tool outputs: Structured JSON with error codes
- Validation: Schema at `schemas/input.json`
```

### Key Principle
> "Garbage in, garbage out" - Quality of perception directly limits quality of reasoning.

---

## 2. Reasoning System

**Purpose:** Analyze information, make decisions, plan actions.

### Reasoning Patterns

| Pattern | When to Use | Example |
|---------|-------------|---------|
| **Direct** | Simple, well-defined tasks | "Format this JSON" |
| **Chain-of-Thought** | Multi-step logic | "Debug this error" |
| **ReAct** | Dynamic, tool-dependent | "Find and fix the bug" |
| **Planning** | Complex, multi-phase | "Implement this feature" |

### Decision Types
1. **What action to take** - Tool selection, response generation
2. **When to stop** - Success criteria evaluation
3. **How to recover** - Error handling, fallback strategies
4. **What to delegate** - Sub-agent spawning decisions

### PRP Application
```markdown
## Reasoning Requirements
- Decision pattern: ReAct (reason → act → observe → repeat)
- Stop condition: All tests pass + lint clean
- Fallback: On 3 failures, surface to user
- Delegation: Spawn codebase_analyst for pattern discovery
```

---

## 3. Memory System

**Purpose:** Maintain state across interactions and actions.

### Memory Types

| Type | Scope | Lifetime | Use Case |
|------|-------|----------|----------|
| **Working** | Current turn | Milliseconds | Active reasoning |
| **Session** | Conversation | Hours | Multi-turn context |
| **Persistent** | Cross-session | Days/Weeks | User preferences |
| **Artifacts** | External files | Permanent | Code, documents |

### Memory Operations
- **Write** - Store new information
- **Read** - Retrieve relevant context
- **Update** - Modify existing state
- **Forget** - Compress or remove stale data

### PRP Application
```markdown
## Memory Requirements
- Session state: Track completed tasks in conversation
- Persistent: User's codebase patterns in `~/.prp-cache/`
- Artifacts: Generated PRPs saved to `PRPs/` directory
- Compression: Summarize completed tasks after validation
```

See `memory-architecture.md` for the four-layer model.

---

## 4. Execution System

**Purpose:** Take actions and interact with external world.

### Execution Capabilities
- **Tool invocation** - Call external functions/APIs
- **Code execution** - Run scripts, tests, builds
- **File operations** - Read, write, modify files
- **Communication** - Send messages, notifications

### Execution Principles

1. **Validate before execute** - Check parameters, permissions
2. **Atomic operations** - One action per step when possible
3. **Capture outputs** - All results feed back to perception
4. **Handle failures** - Every action needs error path

### PRP Application
```markdown
## Execution Requirements
- Tools available: Bash, Read, Write, Glob, Grep
- Validation: `pytest` after each code change
- Rollback: `git stash` before risky operations
- Timeout: 120s max per tool invocation
```

---

## System Interactions

### The Agent Loop

```
User Input
    │
    ▼
┌─────────────┐
│ PERCEPTION  │ ← Parse input, load context
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  REASONING  │ ← Decide action
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  EXECUTION  │ ← Take action
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   MEMORY    │ ← Update state
└──────┬──────┘
       │
       ▼
   Loop or Respond
```

### System Dependencies

| From | To | Data Flow |
|------|-----|-----------|
| Perception → Reasoning | Parsed inputs, retrieved context |
| Reasoning → Execution | Selected actions, parameters |
| Execution → Perception | Tool outputs, results |
| Memory ↔ All | State read/write operations |

---

## Design Implications for PRPs

### 1. Perception Section
```markdown
## Input Handling
- Parse: Extract feature name, scope, constraints from user description
- Context: Load existing patterns from codebase
- Validate: Ensure all required information present
```

### 2. Reasoning Section
```markdown
## Decision Flow
- IF scope unclear → ASK user for clarification
- IF similar feature exists → REFERENCE existing pattern
- IF new domain → USE planning PRP first
```

### 3. Execution Section
```markdown
## Tasks
### Task 1: CREATE `src/feature.py`
- IMPLEMENT: Core functionality
- VALIDATE: `pytest tests/test_feature.py -v`
- IF_FAIL: Check imports, verify dependencies
```

### 4. Memory Section
```markdown
## State Tracking
- Track: Completed tasks, discovered patterns, gotchas
- Persist: Add new patterns to project conventions
- Compress: Summarize tool outputs after processing
```

---

## Common Failure Modes

| System | Failure | Fix |
|--------|---------|-----|
| Perception | Missing context | Explicit context requirements in PRP |
| Reasoning | Infinite loops | Clear stop conditions, max iterations |
| Memory | Context overflow | Aggressive compression, sub-agents |
| Execution | Silent failures | Mandatory validation after each task |

---

## Quick Reference

When building PRPs, ensure each system is addressed:

- [ ] **Perception** - What inputs/context does the agent need?
- [ ] **Reasoning** - What decisions must be made?
- [ ] **Memory** - What state must be tracked?
- [ ] **Execution** - What actions with what validation?
