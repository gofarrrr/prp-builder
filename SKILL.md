# PRP Builder Skill

Build Production-Ready PRPs (Product Requirement Prompts) through guided conversation.

## Description

An interactive skill that guides users through creating comprehensive PRPs using battle-tested templates and context engineering best practices. Produces implementation-ready specifications that AI coding agents can execute with minimal ambiguity.

## Trigger Phrases

- "help me create a PRP"
- "build a PRP for"
- "I need a product requirement prompt"
- "create implementation specs for"
- "/prp-build"

---

## What You Should Know First

Before diving in, here's what PRPs are and when each type fits:

### What is a PRP?

A **Product Requirement Prompt** bridges traditional PRDs with AI-native specifications:
- **PRD** tells *what* and *why* (human-focused)
- **PRP** adds *how*, *where*, and *validation* (agent-focused)

PRPs work because LLMs generate higher-quality code when given direct, in-prompt references rather than broad descriptions.

### Template Selection Guide

| If your work is... | Use | Why |
|--------------------|-----|-----|
| Single file, < 1 hour | **Task PRP** | Minimal overhead, fast execution |
| Sprint task, 2-5 files | **Story PRP** | Structured tasks with validation |
| Major feature, new system | **Base PRP** | Comprehensive context, full validation |
| Unclear scope, needs research | **Planning PRP** | Research before committing to approach |
| LLM/AI integration | **Generative AI PRP** | AI-specific patterns, cost controls |

### Common Mistakes (And How We'll Avoid Them)

| Mistake | Why It Fails | What We'll Do Instead |
|---------|--------------|----------------------|
| Vague references | Agents waste tokens discovering what you know | Request specific file:line references |
| Missing validation | Silent failures propagate | Add VALIDATE after every task |
| Under-specification | Agents fill gaps with assumptions | Match template to actual complexity |
| Over-specification | Competes for attention budget | Keep it concise, reference don't copy |
| Implicit dependencies | Wrong execution order | Explicit task dependencies |

### The Conversation Ahead

I'll guide you through:
1. **Scope Discovery** - What are we building? How complex?
2. **Context Gathering** - Who, what, why, constraints
3. **Codebase Analysis** - Discover patterns (if applicable)
4. **PRP Generation** - Build the specification
5. **Review & Validation** - Ensure completeness

Along the way, I'll:
- **Suggest** better approaches when I see opportunities
- **Push back** when choices don't match the situation
- **Explain** why certain practices matter
- **Ask** clarifying questions before assuming

Ready? Let's start.

---

## Interactive Flow

When activated, follow this conversational flow to gather requirements and build the PRP:

### Phase 1: Scope Discovery

Ask the user:

```
What are we building today?

I'll help you create a PRP (Product Requirement Prompt) - a specification
that gives AI coding agents everything needed to implement your feature.

To get started, tell me:
1. **What's the feature/task?** (one sentence)
2. **What type of work is this?**
   - [ ] New feature (greenfield)
   - [ ] Enhancement to existing feature
   - [ ] Bug fix
   - [ ] Refactoring
   - [ ] Sprint task from Jira/Linear
3. **Scope estimate:**
   - [ ] Small (single file, < 1 hour manual work)
   - [ ] Medium (2-5 files, partial day)
   - [ ] Large (system-wide, multiple days)
```

Based on response, select PRP type:
- Small + Bug fix/Sprint task → **Task PRP** (lightweight)
- Medium + Enhancement/Feature → **Story PRP** (tactical)
- Large + New feature → **Base PRP** (comprehensive)
- Any + Needs research first → **Planning PRP** (research-first)
- Any + LLM/AI integration → **Generative AI PRP** (AI-specialized)

### Phase 2: Context Gathering

#### 2A. For Task/Story PRPs:

```
Let me understand the context:

1. **Which codebase area?** (paste file paths or describe location)
2. **Existing patterns to follow?** (or should I discover them?)
3. **Acceptance criteria?** (what must be true when done)
4. **Known constraints?** (libraries, performance, compatibility)
```

#### 2B. For Base/Planning PRPs:

```
Let's build a complete picture:

**User Context:**
- Who uses this feature?
- What's their workflow/pain point?
- What does success look like for them?

**Technical Context:**
- Tech stack constraints?
- Integration points with existing systems?
- Performance/security requirements?

**Business Context:**
- Why now? What problem does this solve?
- Dependencies on other work?
```

#### 2C. For Generative AI PRPs:

```
Let's understand your AI integration:

**LLM Configuration:**
- Which provider(s)? (Claude, OpenAI, both, other)
- Model requirements? (speed vs quality trade-off)
- Context window needs? (short prompts vs long documents)

**Prompt Engineering:**
- New prompts needed or extending existing?
- Few-shot examples required?
- Prompt chaining/multi-step reasoning?

**Infrastructure:**
- Caching strategy? (reduce costs/latency)
- Rate limiting requirements?
- Token budget constraints?

**Quality & Cost:**
- Response quality threshold?
- Cost per request budget?
- Monitoring requirements?
```

### Phase 3: Codebase Analysis (if applicable)

If user provides file paths or we're in a project:

```
I'll analyze your codebase to discover:
- Existing patterns and conventions
- Related implementations to reference
- Testing patterns in use
- Potential gotchas

[Run codebase analysis using @codebase-analyst patterns]
```

Use these discovery commands:
```bash
# Find similar implementations
rg -l "similar_pattern" --type py

# Discover test patterns
fd -e test.py -e _test.py | head -10

# Check existing models/schemas
rg "class.*Model|class.*Schema" --type py
```

### Phase 4: PRP Generation

Based on gathered context, generate the appropriate PRP:

#### For Task PRP:
```markdown
# Task: [TITLE]

## Context
- **File(s):** `path/to/file.py`
- **Pattern:** Follow existing `similar_feature.py`
- **Gotcha:** [discovered constraint]

## Tasks

### Task 1: [ACTION] `path/to/file.py`
- IMPLEMENT: [specific change]
- PATTERN: Reference `existing_pattern.py:45-60`
- IMPORTS: [required imports]
- VALIDATE: `pytest path/to/test_file.py -v`
- IF_FAIL: [debug hint]

## Completion Checklist
- [ ] All tasks validated
- [ ] Tests pass
- [ ] Follows existing patterns
```

#### For Story PRP:
Use `references/story-template.md` structure with:
- Story foundation + metadata
- Auto-discovered patterns
- Implementation tasks with validation loops
- Four-level validation framework

#### For Base PRP:
Use `references/base-template.md` structure with:
- Goal/User/Why/What sections
- Context completeness check
- Documentation references (YAML format)
- Codebase trees (current → desired)
- Implementation patterns with inline gotchas
- Integration points
- Four-level validation
- Anti-patterns section

#### For Planning PRP:
Use `templates/planning.md` with:
- Phase 1: Research (market, technical, internal)
- Phase 2: PRD structure with diagrams
- Phase 3: Risk analysis + edge cases
- Phase 4: Validation + handoff

#### For Generative AI PRP:
Use `templates/generative-ai.md` with:
- Reference architecture (config/, src/llm/, src/prompt_engineering/, etc.)
- Key components mapping to standard structure
- AI-specific best practices checklist
- Phased implementation (Config → LLM Client → Prompts → Utils → Handlers)
- AI-specific validation (token budgets, response quality, cost estimation)
- Provider-specific gotchas (Claude vs OpenAI differences)
- Cost control and quality control sections

### Phase 5: Review & Refinement

```
Here's your PRP draft. Let's validate it:

**Context Completeness Check:**
Could another developer implement this using ONLY this PRP?
- [ ] All file paths specified
- [ ] Patterns referenced with line numbers
- [ ] Dependencies listed
- [ ] Gotchas documented

**What's missing or unclear?**
```

Iterate until the user confirms completeness.

---

## Best Practices Guidance

During PRP creation, explain what's:

### Essential (Must Have)
- **Clear acceptance criteria** - Without these, you can't validate completion
- **Specific file paths** - Vague references waste tokens on discovery
- **Validation commands** - Executable checks prevent silent failures
- **Pattern references** - "Follow X" beats describing conventions

### Recommended (Should Have)
- **Gotcha documentation** - Prevents common mistakes
- **Context completeness check** - Self-validates the PRP
- **Codebase trees** - Visual clarity on scope
- **Four-level validation** - Catches issues at appropriate stages

### Optional (Nice to Have)
- **Mermaid diagrams** - Helpful for complex flows
- **Alternative approaches** - When trade-offs matter
- **Performance benchmarks** - For optimization work
- **Rollback procedures** - For risky changes

### Overkill (Skip Unless Needed)
- **Exhaustive edge cases** - Focus on likely scenarios first
- **Multi-phase migrations** - Only for truly large changes
- **External stakeholder sections** - Keep it developer-focused

---

## Context Engineering Integration

When building PRPs, apply these principles:

### Progressive Disclosure
Load context in stages:
1. **Startup:** Skill description + user's initial request
2. **Phase 2:** Relevant template sections only
3. **Phase 3:** Codebase patterns (discovered, not pre-loaded)
4. **Phase 4:** Full template for generation

### Token Budget Awareness
- Keep PRP concise - agents have context limits too
- Use references over inline documentation
- Prefer `PATTERN: file:line` over copying code
- Compress verbose tool outputs in final PRP

### Attention Positioning
- Place critical info (gotchas, constraints) at section boundaries
- Lead tasks with ACTION keywords for scannability
- Put validation commands immediately after each task

### Degradation Prevention
- Single responsibility per task
- Clear dependency ordering
- No contradictory instructions
- Explicit over implicit

---

## File References

Load these when generating PRPs:

| Template Type | Reference File | When to Use |
|---------------|----------------|-------------|
| Task | `templates/task.md` | Single-file changes, bug fixes |
| Story | `templates/story.md` | Sprint tasks, scoped features |
| Base | `templates/base.md` | Major features, new systems |
| Planning | `templates/planning.md` | Research-first, complex scope |
| Generative AI | `templates/generative-ai.md` | LLM integrations, AI features |
| Tasks from PRP | `templates/tasks-from-prp.md` | Generate task checklist from completed PRP |

| Knowledge Base | Purpose |
|----------------|---------|
| `references/context-engineering.md` | Context optimization principles |
| `references/validation-framework.md` | Four-level validation guide |
| `references/anti-patterns.md` | What to avoid in PRPs |
| `references/best-practices-2025.md` | Current recommendations |
| `references/guidance-and-pushback.md` | Proactive guidance triggers |
| `references/multi-agent-patterns.md` | When/how to use multiple agents |

---

## Output Format

Final PRP should be:
1. **Self-contained** - Implementable without additional context
2. **Validated** - Pass the "could another dev implement this?" test
3. **Actionable** - Every section drives toward completion
4. **Scoped** - No feature creep, no over-engineering

Save to: `PRPs/[feature-name]-prp.md`

---

## Example Invocation

**User:** "Help me create a PRP for adding user authentication"

**Response Flow:**
1. Ask scope questions (new feature? which auth method?)
2. Gather context (tech stack, existing user model?)
3. Analyze codebase (discover patterns, find integration points)
4. Generate Base PRP with auth-specific sections
5. Review context completeness
6. Refine based on feedback
7. Output final PRP

---

## Proactive Guidance & Pushback

### Core Philosophy

**Genius Intern Framework:** AI agents need context about quality expectations in YOUR specific situation—not generic best practices.

**Async Remote Teammate:** Write PRPs the way you'd write for a smart remote colleague—clear, comprehensive, and decisively opinionated.

### When to Push Back

**Scope Mismatches:**
```
📊 Scope Check

Your description suggests [complexity level], but you selected [template type].

| Indicator | Your Feature | Template Expectation |
|-----------|--------------|---------------------|
| Files affected | [N] | [expected range] |
| Systems touched | [list] | [expected] |

Recommendation: [template] would provide [specific benefits].
```

**Missing Context:**
```
⚠️ Context Gap Detected

Your PRP is missing [element]:
- Impact: [what goes wrong without it]
- Fix: [how to provide it]

Can you provide this, or should I help discover it?
```

**Over-Engineering:**
```
✂️ Simplification Opportunity

Section [X] has more detail than agents typically need.
Agents work best with high-signal, concise context.
Extra detail competes for attention without adding value.
```

### Guidance Triggers

| Trigger | Response |
|---------|----------|
| "Small" scope + multiple systems | Suggest Story PRP upgrade |
| Vague file references | Request specific paths |
| Missing acceptance criteria | Explain testability need |
| Too much documentation linked | Help filter to essentials |
| Skipping codebase analysis | Warn about pattern conflicts |
| Wrong template insistence | Explain consequences, offer alternative |

### Educational Moments

When making recommendations, explain **why**:
```
I'm recommending [structure] because:

Context engineering principle: [specific principle]
- Your situation: [how it applies]
- Risk if ignored: [what goes wrong]
- Benefit: [what improves]

This isn't arbitrary—it's how agents process information.
```

### Decisiveness Over Hedging

**Don't say:** "You might want to consider possibly using..."
**Do say:** "Use Story PRP. Here's why: [reasons]. Proceed?"

PRPs should be opinionated. Guide users to decisions, don't present endless options.

---

## PRP → Task Generation Workflow

When a PRP is complete and approved, generate trackable task lists:

### Trigger
```
"Generate tasks from this PRP"
"Break this PRP into tasks"
"Create task checklist"
```

### Two-Phase Process

**Phase 1: High-Level Tasks**
```markdown
## Parent Tasks

- [ ] **0.0 Create feature branch**
- [ ] **1.0 [First major component]**
- [ ] **2.0 [Second major component]**
- [ ] **3.0 Testing & validation**
- [ ] **4.0 Integration & cleanup**

*Review these. Reply "Go" to expand into sub-tasks.*
```

**Phase 2: Expanded Sub-Tasks**
```markdown
- [ ] **1.0 [Component name]**
  - [ ] 1.1 [Specific action with file path]
  - [ ] 1.2 [Next action]
  - [ ] 1.3 Validate: `[command]`
```

### Task Tracking
After completing each sub-task:
- Change `- [ ]` to `- [x]`
- Validate before moving to next
- Commit if substantial

### Output Location
```
tasks/tasks-[feature-name].md
# or
PRPs/[feature-name]-tasks.md
```

Use `templates/tasks-from-prp.md` for full format specification.

---

## Complete Workflow

```
User Request
     │
     ▼
┌─────────────────┐
│  Scope Discovery │ ← Select template type
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Context Gather  │ ← Questions + codebase analysis
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ PRP Generation  │ ← Build specification
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Review & Refine │ ← Context completeness check
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Task Generation │ ← Phase 1: Parent tasks
│   (Optional)    │   Phase 2: Sub-tasks
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Implementation  │ ← Check off tasks as complete
└─────────────────┘
```

---

## Integration with Other Skills

- **After PRP creation:** Offer to generate task checklist
- **If stuck on scope:** Suggest `/prp-planning` for research phase
- **For quick tasks:** Skip to Task PRP format directly
- **For implementation:** Work through tasks incrementally with validation
