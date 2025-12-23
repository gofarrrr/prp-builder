# PRP Builder

A Claude Code skill for building Production-Ready PRPs (Product Requirement Prompts) through guided conversation.

PRPs bridge traditional Product Requirements Documents with AI-native specifications—giving coding agents everything they need to implement features with minimal ambiguity.

## What is a PRP?

| Document | Focus | Audience |
|----------|-------|----------|
| **PRD** | *What* and *Why* | Humans |
| **PRP** | *What*, *Why*, *How*, *Where*, and *Validation* | AI Agents |

PRPs work because LLMs generate higher-quality code when given direct, in-prompt references rather than broad descriptions.

## Features

- **Interactive Flow** — Guided conversation to gather requirements
- **5 PRP Templates** — From lightweight tasks to comprehensive features
- **Context Engineering** — Optimized for agent attention budgets
- **Proactive Guidance** — Pushback when choices don't fit the situation
- **Task Generation** — Convert PRPs to trackable checkbox task lists
- **Best Practices 2025** — Current recommendations with necessity levels

## Templates

| Template | Use Case | Complexity |
|----------|----------|------------|
| **Task** | Single-file changes, bug fixes | Small |
| **Story** | Sprint tasks, scoped features | Medium |
| **Base** | Major features, new systems | Large |
| **Planning** | Research-first, unclear scope | Variable |
| **Generative AI** | LLM integrations, AI features | Variable |
| **Tasks from PRP** | Generate task checklist from completed PRP | Post-PRP |

## Installation

### Claude Code

```bash
# Add to your Claude Code skills
claude skill add gofarrrr/prp-builder
```

### Manual Installation

1. Clone this repository
2. Copy `prp-builder/` to your Claude Code skills directory
3. Restart Claude Code

## Usage

### Start Building a PRP

```
/prp-build
```

Or use natural language:
- "Help me create a PRP for user authentication"
- "Build a PRP for adding dark mode"
- "I need implementation specs for the payment integration"

### Generate Tasks from PRP

After your PRP is complete:
```
"Generate tasks from this PRP"
```

This creates a trackable task list with checkboxes:
```markdown
- [ ] **1.0 Authentication Service**
  - [ ] 1.1 Create `src/services/auth_service.py`
  - [ ] 1.2 Implement login method
  - [ ] 1.3 Validate: `pytest tests/unit/test_auth.py -v`
```

## Structure

```
prp-builder/
├── SKILL.md                     # Main skill definition
├── templates/
│   ├── task.md                  # Lightweight template
│   ├── story.md                 # Sprint task template
│   ├── base.md                  # Comprehensive template
│   ├── planning.md              # Research-first template
│   ├── generative-ai.md         # AI/LLM specialized
│   └── tasks-from-prp.md        # Task generation
└── references/
    ├── context-engineering.md   # Token budgets, attention
    ├── validation-framework.md  # Four-level validation
    ├── anti-patterns.md         # What to avoid
    ├── best-practices-2025.md   # Current recommendations
    ├── guidance-and-pushback.md # Proactive intervention
    └── multi-agent-patterns.md  # Multi-agent architecture
```

## Best Practices

### Essential (Must Have)
- **Executable acceptance criteria** — Binary pass/fail checks
- **Specific file references** — `src/auth/jwt.py:45-60`, not "the auth module"
- **Immediate validation** — VALIDATE command after every task
- **Pattern references** — Point to existing code, don't describe

### Recommended (Should Have)
- Context completeness check
- Gotcha documentation
- Codebase trees (current → desired)
- Four-level validation

### Optional (Nice to Have)
- Mermaid diagrams for complex flows
- Alternative approaches with trade-offs
- Performance benchmarks

### Skip Unless Needed
- Exhaustive edge cases
- Multi-phase migration plans
- External stakeholder sections

## Context Engineering Principles

This skill applies context engineering best practices:

- **Progressive Disclosure** — Load information only when needed
- **Token Budget Awareness** — Keep PRPs concise
- **Attention Positioning** — Critical info at section boundaries
- **Degradation Prevention** — Single responsibility per task

## Proactive Guidance

The skill will push back when:

| Situation | Response |
|-----------|----------|
| "Small" scope + multiple systems | Suggest Story PRP upgrade |
| Vague file references | Request specific paths |
| Missing acceptance criteria | Explain testability need |
| Wrong template for complexity | Explain consequences |
| Over-specification | Help simplify |

## Sources & Credits

This skill combines knowledge and patterns from:

| Source | Contribution |
|--------|--------------|
| [Wirasm/PRPs-agentic-eng](https://github.com/Wirasm/PRPs-agentic-eng) | PRP templates, validation framework, story workflow |
| [muratcankoylan/Agent-Skills-for-Context-Engineering](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering) | Context engineering principles, multi-agent patterns |
| [nityeshaga/claude-code-essentials](https://github.com/nityeshaga/claude-code-essentials) | Proactive guidance, tutorial approach, skill structure |
| [snarktank/ai-dev-tasks](https://github.com/snarktank/ai-dev-tasks) | PRD→Tasks workflow, checkbox tracking |
| [Nina Duran's Generative AI Project Structure](https://github.com/heynina101) | AI project architecture template |

## Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Follow existing patterns in templates
4. Submit a pull request

## License

MIT License — see [LICENSE](LICENSE) for details.

---

Built with Claude Code
