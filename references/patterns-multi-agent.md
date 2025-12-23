# Multi-Agent Design Patterns

When to use multiple agents and how to structure their collaboration.

---

## When to Deploy Multi-Agent Systems

Use multiple agents when:
- Single agent's context window constrains task complexity
- Work naturally breaks into parallel subtasks
- Different subtasks need specialized tools/instructions
- Must simultaneously handle multiple domains

**Core insight:** Sub-agents exist to **isolate context**, not to role-play. Each operates in a clean, focused context.

---

## Pattern 1: Supervisor/Orchestrator

**What:** Central coordinator delegates to specialized workers.

```
┌─────────────────────────────────────┐
│           Supervisor                │
│    (maintains global state)         │
└──────────┬──────────┬───────────────┘
           │          │
     ┌─────▼────┐ ┌───▼─────┐
     │ Worker A │ │ Worker B │
     │ (Search) │ │ (Analyze)│
     └──────────┘ └──────────┘
```

**When to Use:**
- Complex tasks with clear decomposition
- Need oversight across domains
- Human intervention points important
- Quality control required

**Supervisor Responsibilities:**
- Parse user request
- Decompose into subtasks
- Assign to appropriate workers
- Synthesize worker outputs
- Handle errors and escalations

**Worker Characteristics:**
- Single, focused capability
- Limited context (just their task)
- Returns structured output
- No awareness of other workers

**PRP Application:**
```yaml
supervisor: "prp_builder_orchestrator"
  responsibilities:
    - "Guide user through requirements gathering"
    - "Select appropriate PRP template"
    - "Coordinate specialist analysis"
    - "Synthesize final PRP"
    - "Validate context completeness"

workers:
  codebase_analyst:
    trigger: "User provides project context"
    tasks:
      - "Glob for similar implementations"
      - "Grep for patterns and conventions"
      - "Read key files for structure"
    output: "Discovered patterns, gotchas, file structure"

  library_researcher:
    trigger: "External libraries mentioned"
    tasks:
      - "Fetch library documentation"
      - "Search for known issues"
      - "Find implementation examples"
    output: "Documentation references, constraints, examples"
```

**Key Challenge: Telephone Game Problem**

Supervisors paraphrasing sub-agent responses lose fidelity.

**Solution:** Forward directly when appropriate:
```python
if response.is_complete and response.confidence > 0.9:
    forward_to_user(response)  # Bypass supervisor
else:
    summarize_for_supervisor(response)
```

---

## Pattern 2: Peer-to-Peer/Swarm

**What:** Agents hand off to each other without central coordinator.

```
┌──────────┐     ┌──────────┐
│ Agent A  │◄────►│ Agent B  │
└────┬─────┘     └─────┬────┘
     │                 │
     └────────┬────────┘
              │
        ┌─────▼─────┐
        │  Agent C  │
        └───────────┘
```

**When to Use:**
- Flexible exploration needed
- Rigid upfront planning counterproductive
- Emergent requirements expected
- No clear hierarchy

**Handoff Protocol:**
```yaml
handoff_protocol:
  - agent: "requirements_gatherer"
    handoff_to: "codebase_analyst"
    trigger: "requirements complete"
    state_passed: ["user_goals", "constraints"]

  - agent: "codebase_analyst"
    handoff_to: "template_selector"
    trigger: "patterns discovered"
    state_passed: ["patterns", "gotchas", "file_structure"]

  - agent: "template_selector"
    handoff_to: "prp_generator"
    trigger: "template chosen"
    state_passed: ["template_type", "populated_sections"]
```

**Advantages:**
- No single point of failure
- Effective for breadth-first exploration
- Enables emergent problem-solving
- Lower coordination overhead

**Disadvantages:**
- Harder to track progress
- Risk of circular handoffs
- No global optimization

---

## Pattern 3: Hierarchical

**What:** Multiple layers of agents with different abstraction levels.

```
┌───────────────────────────────────┐
│         Strategy Layer            │
│      (goal definition)            │
└──────────────┬────────────────────┘
               │
┌──────────────▼────────────────────┐
│         Planning Layer            │
│      (task decomposition)         │
└───────┬───────────────┬───────────┘
        │               │
┌───────▼───────┐ ┌─────▼─────────┐
│  Execution    │ │   Execution   │
│   (atomic)    │ │   (atomic)    │
└───────────────┘ └───────────────┘
```

**When to Use:**
- Large-scale projects
- Enterprise workflows
- Need both high-level planning and detailed execution
- Complex dependency management

**Layer Definitions:**
```yaml
layers:
  strategy:
    role: "Define feature scope and success criteria"
    output: "High-level requirements"
    context: "User goals, business constraints"

  planning:
    role: "Decompose into PRP sections"
    output: "Section outlines with dependencies"
    context: "Strategy output, template structure"

  execution:
    role: "Populate each section"
    output: "Completed PRP sections"
    context: "Planning output, specific task"
```

---

## Token Economics

Multi-agent systems consume **significantly more tokens**:

| Query Type | Token Multiplier |
|------------|-----------------|
| Simple query | 1× baseline |
| Complex coordination | ~15× baseline |
| Deep hierarchical | ~25× baseline |

**Key insight:** Model upgrades often yield larger performance gains than simply doubling token budgets.

**Cost Control Strategies:**
```yaml
cost_optimization:
  - "Start with single agent, spawn only when needed"
  - "Use cheaper models for worker agents"
  - "Cache worker outputs aggressively"
  - "Compress handoff state to minimum"
  - "Set max_iterations for all loops"
```

---

## Critical Success Factors

### 1. Output Validation Between Agents

```python
def validate_handoff(source_agent, target_agent, payload):
    # Prevent error propagation
    if not payload.is_valid():
        return request_clarification(source_agent)

    # Ensure completeness
    if payload.missing_required_fields():
        return request_completion(source_agent)

    return proceed(target_agent, payload)
```

### 2. Weighted Voting Over Simple Consensus

```python
# Don't let weak model hallucinations dominate
def aggregate_findings(agent_outputs):
    weighted_results = []
    for output in agent_outputs:
        weight = output.confidence * agent_reliability_score
        weighted_results.append((output, weight))
    return select_highest_weighted(weighted_results)
```

### 3. Time-to-Live Limits

```python
# Prevent infinite loops
MAX_AGENT_HOPS = 5
current_hops = 0

def handoff(target_agent):
    global current_hops
    current_hops += 1
    if current_hops > MAX_AGENT_HOPS:
        return escalate_to_user()
    return execute(target_agent)
```

### 4. Explicit Handoff Protocols

```yaml
handoff:
  from: "codebase_analyst"
  to: "prp_generator"
  state:
    patterns: "[list of discovered patterns]"
    gotchas: "[list of gotchas]"
    file_tree: "[current structure]"
  context_compression: "summarize older findings"
  validation: "all required fields present"
```

---

## Context Isolation Strategy

Each agent gets focused context:

```yaml
context_budgets:
  supervisor:
    always_loaded:
      - "User conversation"
      - "Current PRP draft"
      - "Template structure (compressed)"
    loaded_on_demand:
      - "Specialist outputs (summarized)"
    never_loaded:
      - "Raw tool outputs from workers"
      - "Full file contents"

  codebase_analyst:
    always_loaded:
      - "Current task"
      - "File patterns to search"
    loaded_on_demand:
      - "File contents (on read)"
    never_loaded:
      - "Full user conversation history"
      - "Library documentation"

  library_researcher:
    always_loaded:
      - "Library names and versions"
      - "Specific questions"
    never_loaded:
      - "Codebase details"
      - "User conversation history"
```

---

## When NOT to Use Multi-Agent

Single agent is better when:
- Task fits comfortably in one context window
- No clear parallelization opportunity
- Coordination overhead exceeds benefit
- Simple, well-defined task
- Latency is critical

**Decision Matrix:**

| Task Characteristic | Single Agent | Multi-Agent |
|---------------------|--------------|-------------|
| Files affected: 1-3 | ✓ | |
| Files affected: 10+ | | ✓ |
| Clear steps | ✓ | |
| Exploration needed | | ✓ |
| Time-sensitive | ✓ | |
| Quality-critical | | ✓ |
| Known domain | ✓ | |
| Unknown domain | | ✓ |

---

## Multi-Agent for PRP Builder

**Recommended Architecture:**

For most PRPs, use **single agent + tool calls**. Reserve multi-agent for:
- Large codebases requiring extensive analysis
- Features spanning multiple unfamiliar domains
- Planning PRPs with significant research phase

**When to Spawn:**

```yaml
spawn_triggers:
  - condition: "codebase > 100 files"
    spawn: "codebase_analyst"
    reason: "Heavy file loading isolated"

  - condition: "unfamiliar libraries mentioned"
    spawn: "library_researcher"
    reason: "External fetching isolated"

  - condition: "planning PRP selected"
    spawn: "research_agent"
    reason: "Broad exploration needed"
```

---

## Multi-Agent Checklist

Before implementing multi-agent:

- [ ] Is single agent insufficient? (try it first)
- [ ] Are agent responsibilities clearly separated?
- [ ] Is handoff protocol defined?
- [ ] Are TTL limits set?
- [ ] Is output validation in place?
- [ ] Is context isolation configured?
- [ ] Are costs acceptable?
- [ ] Is escalation path defined?
