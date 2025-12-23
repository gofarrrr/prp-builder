# Parallel Design Patterns

Patterns for concurrent agent workflows and result aggregation.

---

## Overview

Parallel patterns execute multiple operations simultaneously, then aggregate results.

```
        Input
          │
    ┌─────┼─────┐
    │     │     │
    ▼     ▼     ▼
  Task  Task  Task
    A     B     C
    │     │     │
    └─────┼─────┘
          │
          ▼
     Aggregation
          │
          ▼
       Output
```

---

## 1. Sectioning (Divide & Conquer)

**What:** Split input into independent sections, process each separately.

**When to Use:**
- Task naturally divides into parts
- Parts are independent
- Results can be combined

**Pattern:**
```
Large Task
    │
    ▼
┌───────────┐
│  DIVIDE   │
└─────┬─────┘
      │
  ┌───┼───┐
  │   │   │
  ▼   ▼   ▼
┌───┐┌───┐┌───┐
│ A ││ B ││ C │  ← Process in parallel
└─┬─┘└─┬─┘└─┬─┘
  │   │   │
  └───┼───┘
      │
      ▼
┌───────────┐
│  COMBINE  │
└─────┬─────┘
      │
      ▼
   Result
```

**PRP Application:**
```markdown
## Codebase Analysis (Sectioned)

### Division Strategy
Split codebase by domain:
- Section A: `src/services/` - Business logic patterns
- Section B: `src/api/` - API route patterns
- Section C: `src/models/` - Data model patterns

### Parallel Processing
```yaml
analyses:
  - section: services
    agent: codebase_analyst_1
    focus: "Service patterns, base classes"

  - section: api
    agent: codebase_analyst_2
    focus: "Route patterns, middleware"

  - section: models
    agent: codebase_analyst_3
    focus: "Model patterns, relationships"
```

### Combination
Merge findings into unified context:
- Patterns: [merged from all sections]
- Gotchas: [deduplicated across sections]
- File tree: [complete structure]
```

---

## 2. Map-Reduce

**What:** Apply same operation to many items (map), then aggregate results (reduce).

**When to Use:**
- Same operation on multiple inputs
- Results need aggregation
- Order doesn't matter

**Pattern:**
```
Items: [A, B, C, D]
          │
          ▼
┌─────────────────┐
│       MAP       │
│  apply(f, item) │
└────────┬────────┘
         │
    ┌────┼────┬────┐
    │    │    │    │
    ▼    ▼    ▼    ▼
  f(A) f(B) f(C) f(D)
    │    │    │    │
    └────┼────┴────┘
         │
         ▼
┌─────────────────┐
│     REDUCE      │
│  aggregate()    │
└────────┬────────┘
         │
         ▼
     Result
```

**PRP Application:**
```markdown
## Multi-File Validation (Map-Reduce)

### Map Phase
For each file in changeset:
```python
def validate_file(file_path):
    return {
        "file": file_path,
        "lint": run_lint(file_path),
        "type_check": run_mypy(file_path),
        "tests": run_tests(file_path)
    }
```

### Files to Process
- `src/services/auth.py`
- `src/api/routes/auth.py`
- `src/models/user.py`
- `tests/test_auth.py`

### Reduce Phase
```python
def aggregate_results(results):
    return {
        "all_passed": all(r["lint"] and r["type_check"] for r in results),
        "test_summary": summarize_tests(results),
        "issues": [r for r in results if not r["lint"]]
    }
```

### Output
- Overall status: PASS/FAIL
- Per-file results
- Aggregated issues list
```

---

## 3. Voting (Consensus)

**What:** Multiple agents/approaches vote on answer, select best.

**When to Use:**
- Uncertain outcomes
- Multiple valid approaches
- Quality through diversity

**Pattern:**
```
Question
    │
    ▼
┌───────────────┐
│   DISPATCH    │
└───────┬───────┘
        │
   ┌────┼────┐
   │    │    │
   ▼    ▼    ▼
┌────┐┌────┐┌────┐
│Vote││Vote││Vote│  ← Different approaches
│ A  ││ B  ││ C  │
└──┬─┘└──┬─┘└──┬─┘
   │     │     │
   └─────┼─────┘
         │
         ▼
┌─────────────────┐
│   AGGREGATE     │
│  select_best()  │
└────────┬────────┘
         │
         ▼
     Winner
```

**Voting Strategies:**

| Strategy | When to Use |
|----------|-------------|
| Majority | Simple consensus |
| Weighted | Agents have different reliability |
| Unanimous | High-stakes decisions |
| Best-of-N | Quality optimization |

**PRP Application:**
```markdown
## Template Selection (Voting)

### Voters
1. **Complexity Analyzer**
   - Counts files, dependencies, systems
   - Votes based on quantitative metrics

2. **Scope Classifier**
   - Analyzes user description
   - Votes based on language patterns

3. **Historical Matcher**
   - Compares to past PRPs
   - Votes based on similarity

### Voting Logic
```python
def select_template(votes):
    weighted_votes = {
        votes["complexity"]: 0.4,  # 40% weight
        votes["scope"]: 0.35,      # 35% weight
        votes["historical"]: 0.25  # 25% weight
    }
    return max(weighted_votes, key=weighted_votes.get)
```

### Consensus Requirement
- IF 2/3 agree → Use consensus choice
- IF no consensus → Present options to user
```

---

## 4. Parallelization

**What:** Execute independent operations concurrently.

**When to Use:**
- Operations have no dependencies
- Time savings significant
- Resources available

**Pattern:**
```
Independent Tasks
    │
    ├─────────────────┐
    │                 │
    ▼                 ▼
┌─────────┐     ┌─────────┐
│ Task A  │     │ Task B  │  ← Run simultaneously
│ (3 sec) │     │ (5 sec) │
└────┬────┘     └────┬────┘
     │               │
     └───────┬───────┘
             │
             ▼
        Both done
        (5 sec total, not 8)
```

**PRP Application:**
```markdown
## Parallel Context Gathering

### Independent Operations
These can run simultaneously:

1. **Codebase Search** (no dependencies)
   ```bash
   glob "src/**/*.py"
   grep "class.*Service"
   ```

2. **Library Documentation** (no dependencies)
   ```bash
   fetch "https://docs.lib.com/api"
   ```

3. **Config Analysis** (no dependencies)
   ```bash
   read "pyproject.toml"
   read "config/settings.yaml"
   ```

### Execution
```yaml
parallel_tasks:
  - id: codebase
    commands: [glob, grep]

  - id: docs
    commands: [web_fetch]

  - id: config
    commands: [read, read]

await: all
```

### Merge Results
After all complete:
- Combine codebase patterns
- Add documentation insights
- Include config constraints
```

---

## 5. Speculative Execution

**What:** Start likely-needed work before confirmed, discard if wrong.

**When to Use:**
- High confidence in prediction
- Cost of speculation < cost of waiting
- Results are discardable

**Pattern:**
```
                Input
                  │
         ┌───────┴───────┐
         │               │
         ▼               ▼
┌─────────────┐   ┌─────────────┐
│  Determine  │   │ Speculate   │
│  next step  │   │ likely path │
└──────┬──────┘   └──────┬──────┘
       │                 │
       │          (work ahead)
       │                 │
       ▼                 │
   Decision ─────────────┤
       │                 │
       ├── matches ──────┘ → Use speculation
       │
       └── differs → Discard, compute fresh
```

**PRP Application:**
```markdown
## Speculative Template Loading

### Speculation Strategy
While gathering requirements:
- Speculatively load most common template (Story)
- Speculatively search for common patterns

### Usage
```yaml
speculation:
  - trigger: "user mentions feature"
    speculate: load_template("story.md")
    confidence: 0.7

  - trigger: "user mentions bug"
    speculate: load_template("task.md")
    confidence: 0.85
```

### Outcome
- IF speculation correct → Save ~2 seconds
- IF speculation wrong → Discard, load correct template

### Cost-Benefit
Speculation cost: ~100 tokens
Wait cost: ~2 seconds latency
Accuracy: ~70% for experienced users
→ Worth speculating
```

---

## Aggregation Strategies

How to combine parallel results:

### Concatenation
```python
# Simple merge
results = [task_a_result, task_b_result, task_c_result]
combined = "\n".join(results)
```

### Deduplication
```python
# Merge with dedup
all_patterns = set()
for result in results:
    all_patterns.update(result["patterns"])
```

### Prioritization
```python
# Keep best from each
combined = {
    "patterns": highest_confidence(pattern_results),
    "gotchas": unique_gotchas(gotcha_results),
    "structure": merge_trees(structure_results)
}
```

### Conflict Resolution
```python
# Handle contradictions
if results_conflict(result_a, result_b):
    return ask_user_to_resolve(result_a, result_b)
```

---

## Parallel Pattern Checklist

When designing parallel workflows:

- [ ] Tasks are truly independent?
- [ ] Aggregation strategy is defined?
- [ ] Conflicts are handled?
- [ ] Partial failures don't block everything?
- [ ] Token budget accounts for parallel calls?
- [ ] Results can be deduplicated?
- [ ] Timeout per parallel task is set?
