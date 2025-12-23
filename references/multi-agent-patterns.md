# Multi-Agent Patterns for PRP Development

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

### When to Use
- Complex tasks with clear decomposition
- Need oversight across domains
- Human intervention points important

### For PRP Development
```yaml
supervisor_tasks:
  - "Gather user requirements"
  - "Select appropriate PRP template"
  - "Coordinate specialist agents"
  - "Synthesize final PRP"

specialist_agents:
  codebase_analyst:
    - "Discover existing patterns"
    - "Find similar implementations"
    - "Extract testing conventions"

  library_researcher:
    - "Fetch external documentation"
    - "Identify known gotchas"
    - "Find implementation examples"
```

### Key Challenge: Telephone Game Problem

Supervisors paraphrasing sub-agent responses lose fidelity.

**Solution:** Implement `forward_message` tool for direct responses:
```python
# Let specialist response bypass supervisor when appropriate
if response.is_complete and response.confidence > 0.9:
    forward_to_user(response)
else:
    summarize_for_supervisor(response)
```

---

## Pattern 2: Peer-to-Peer/Swarm

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

### When to Use
- Flexible exploration needed
- Rigid upfront planning counterproductive
- Emergent requirements expected

### For PRP Development
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

### Advantages
- No single point of failure
- Effective for breadth-first exploration
- Enables emergent problem-solving

---

## Pattern 3: Hierarchical

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

### When to Use
- Large-scale projects
- Enterprise workflows
- Need both high-level planning and detailed execution

### For PRP Development
```yaml
layers:
  strategy:
    role: "Define feature scope and success criteria"
    output: "High-level requirements"

  planning:
    role: "Decompose into PRP sections"
    output: "Section outlines with dependencies"

  execution:
    role: "Populate each section"
    output: "Completed PRP sections"
```

---

## Token Economics Reality

Multi-agent systems consume **significantly more tokens**:

| Query Type | Token Multiplier |
|------------|-----------------|
| Simple query | 1× baseline |
| Complex coordination | ~15× baseline |
| Deep hierarchical | ~25× baseline |

**Key insight:** Model upgrades often yield larger performance gains than simply doubling token budgets.

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
```

---

## Application to PRP Builder Skill

### Recommended Architecture

For the PRP Builder, use **Supervisor pattern** with specialized workers:

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

  template_selector:
    trigger: "Requirements gathered"
    tasks:
      - "Assess scope and complexity"
      - "Match to template type"
      - "Identify required sections"
    output: "Template selection with rationale"
```

### Context Isolation Strategy

```yaml
context_budgets:
  supervisor:
    always_loaded:
      - "User conversation"
      - "Current PRP draft"
      - "Template structure (compressed)"
    loaded_on_demand:
      - "Specialist outputs (summarized)"

  codebase_analyst:
    always_loaded:
      - "Current task"
      - "File patterns to search"
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

**For PRP Builder:** Most PRPs can be built with single-agent + tool calls. Reserve multi-agent for:
- Large codebases requiring extensive analysis
- Features spanning multiple unfamiliar domains
- Planning PRPs with significant research phase
