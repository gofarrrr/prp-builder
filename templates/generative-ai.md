# Generative AI Project PRP Template

Specialized template for building scalable Generative AI applications with production-ready architecture.

> Based on Nina Duran's Generative AI Project Structure (github.com/heynina101)

---

## Feature: [AI FEATURE NAME]

> [One-line description of the AI capability being built]

### Metadata
- **Type:** `new-ai-feature` | `llm-integration` | `prompt-engineering` | `optimization`
- **Complexity:** `small` | `medium` | `large`
- **LLM Provider:** `claude` | `openai` | `multi-provider` | `custom`

---

## Goal

### Feature Goal
[What AI capability will exist when complete]

### Deliverable
[Specific outputs: API endpoint, CLI tool, notebook, service, etc.]

### Success Definition
- [ ] [Measurable AI performance criterion]
- [ ] [Latency/cost target]
- [ ] [Quality/accuracy threshold]

---

## Reference Architecture

### Standard Generative AI Project Structure
```
generative_ai_project/
├── config/                          # Configuration outside code
│   ├── __init__.py
│   ├── model_config.yaml            # Model parameters, API keys ref
│   ├── prompt_templates.yaml        # Versioned prompt templates
│   └── logging_config.yaml          # Logging configuration
│
├── src/
│   ├── llm/                         # LLM client abstractions
│   │   ├── __init__.py
│   │   ├── base.py                  # Abstract base client
│   │   ├── claude_client.py         # Anthropic Claude implementation
│   │   ├── gpt_client.py            # OpenAI GPT implementation
│   │   └── utils.py                 # Shared LLM utilities
│   │
│   ├── prompt_engineering/          # Prompt management
│   │   ├── __init__.py
│   │   ├── templates.py             # Template loading/rendering
│   │   ├── few_shot.py              # Few-shot example management
│   │   └── chainer.py               # Prompt chaining logic
│   │
│   ├── utils/                       # Cross-cutting utilities
│   │   ├── __init__.py
│   │   ├── rate_limiter.py          # API rate limiting
│   │   ├── token_counter.py         # Token counting/budgeting
│   │   ├── cache.py                 # Response caching
│   │   └── logger.py                # Structured logging
│   │
│   └── handlers/                    # Error handling
│       ├── __init__.py
│       └── error_handler.py         # Custom exception handling
│
├── data/                            # Data repository
│   ├── cache/                       # Cached responses
│   ├── prompts/                     # Prompt assets
│   ├── outputs/                     # Generated outputs
│   └── embeddings/                  # Vector embeddings
│
├── examples/                        # Reference implementations
│   ├── basic_completion.py          # Simple completion example
│   ├── chat_session.py              # Multi-turn conversation
│   └── chain_prompts.py             # Prompt chaining example
│
├── notebooks/                       # Interactive development
│   ├── prompt_testing.ipynb         # Prompt iteration
│   ├── response_analysis.ipynb      # Output analysis
│   └── model_experimentation.ipynb  # Model comparison
│
├── requirements.txt                 # Package dependencies
├── setup.py                         # Package setup
├── README.md                        # Project documentation
└── Dockerfile                       # Container configuration
```

---

## Key Components Mapping

Map your feature to the architecture:

| Component | Your Implementation | Notes |
|-----------|---------------------|-------|
| `config/model_config.yaml` | [Model settings needed] | |
| `config/prompt_templates.yaml` | [New templates] | |
| `src/llm/` | [Client to use/create] | |
| `src/prompt_engineering/` | [Prompt logic needed] | |
| `src/utils/` | [Utilities needed] | |
| `data/` | [Data artifacts] | |

---

## Best Practices Checklist

### Essential for AI Projects
- [ ] **Track prompt versions and results** - Version control all prompts
- [ ] **Structure code by clear module boundaries** - Separate concerns
- [ ] **Cache responses to reduce latency and cost** - Implement caching
- [ ] **Handle errors with custom exceptions** - AI-specific error handling
- [ ] **Use notebooks for rapid testing and iteration** - Prototype first
- [ ] **Monitor API usage and set rate limits** - Cost control

### Development Tips
- [ ] Use modular structure
- [ ] Test components early
- [ ] Track with version control
- [ ] Keep datasets fresh
- [ ] Monitor API usage

---

## Implementation Tasks

### Phase 1: Configuration Setup

#### Task 1: UPDATE `config/model_config.yaml`
```yaml
# Add model configuration for your feature
[feature_name]:
  model: "claude-3-5-sonnet-20241022"  # or gpt-4, etc.
  max_tokens: 4096
  temperature: 0.7
  # Feature-specific params
```
- **VALIDATE:** `python -c "import yaml; yaml.safe_load(open('config/model_config.yaml'))"`
- **IF_FAIL:** Check YAML syntax, validate against schema

#### Task 2: CREATE/UPDATE `config/prompt_templates.yaml`
```yaml
# Version your prompts
[feature_name]_v1:
  system: |
    [System prompt content]
  user: |
    [User prompt template with {variables}]
  version: "1.0.0"
  created: "[date]"
```
- **VALIDATE:** Template renders with test variables
- **IF_FAIL:** Check variable placeholders, escape special chars

### Phase 2: LLM Client Layer

#### Task 3: [CREATE|UPDATE] `src/llm/[provider]_client.py`
- **IMPLEMENT:** Client method for feature
- **PATTERN:** `src/llm/base.py` abstract interface
- **IMPORTS:** Provider SDK, config loader, error handlers
- **GOTCHA:** Handle rate limits, implement retries
- **VALIDATE:** `pytest tests/unit/test_llm_client.py -v`

#### Task 4: UPDATE `src/llm/utils.py` (if needed)
- **IMPLEMENT:** Shared utilities for feature
- **PATTERN:** Existing utility functions
- **VALIDATE:** `pytest tests/unit/test_llm_utils.py -v`

### Phase 3: Prompt Engineering

#### Task 5: [CREATE|UPDATE] `src/prompt_engineering/templates.py`
- **IMPLEMENT:** Template loading/rendering for feature
- **PATTERN:** Existing template patterns
- **GOTCHA:** Handle variable injection safely
- **VALIDATE:** `pytest tests/unit/test_templates.py -v`

#### Task 6: [CREATE|UPDATE] `src/prompt_engineering/[feature].py`
- **IMPLEMENT:** Feature-specific prompt logic
- **OPTIONS:**
  - `few_shot.py` - If using few-shot examples
  - `chainer.py` - If chaining multiple prompts
  - New file for unique logic
- **VALIDATE:** Unit tests pass

### Phase 4: Utilities Integration

#### Task 7: UPDATE `src/utils/rate_limiter.py`
- **IMPLEMENT:** Rate limiting for new endpoints
- **PATTERN:** Existing rate limiter patterns
- **GOTCHA:** Different providers have different limits
- **VALIDATE:** Rate limiting activates correctly

#### Task 8: UPDATE `src/utils/token_counter.py`
- **IMPLEMENT:** Token counting for feature prompts
- **PATTERN:** Provider-specific tokenizers
- **GOTCHA:** Claude vs GPT tokenization differs
- **VALIDATE:** Token counts match API response usage

#### Task 9: UPDATE `src/utils/cache.py`
- **IMPLEMENT:** Caching strategy for feature
- **CONSIDER:**
  - Cache key generation
  - TTL settings
  - Invalidation strategy
- **VALIDATE:** Cache hits work, stale data handled

### Phase 5: Error Handling

#### Task 10: UPDATE `src/handlers/error_handler.py`
- **IMPLEMENT:** Feature-specific error handling
- **HANDLE:**
  - API rate limits (429)
  - Token limit exceeded
  - Invalid responses
  - Timeout errors
  - Content filtering
- **PATTERN:** Existing error handler
- **VALIDATE:** Error scenarios handled gracefully

### Phase 6: Examples & Documentation

#### Task 11: CREATE `examples/[feature]_example.py`
- **IMPLEMENT:** Working example demonstrating feature
- **PATTERN:** `examples/basic_completion.py`
- **INCLUDE:** Comments explaining each step
- **VALIDATE:** Example runs successfully

#### Task 12: CREATE `notebooks/[feature]_testing.ipynb`
- **IMPLEMENT:** Interactive notebook for feature testing
- **INCLUDE:**
  - Setup cells
  - Prompt testing
  - Response analysis
  - Parameter tuning
- **VALIDATE:** Notebook runs end-to-end

---

## Data Artifacts

### Prompt Storage
```
data/prompts/
└── [feature]/
    ├── system_prompt_v1.txt
    ├── few_shot_examples.json
    └── test_cases.json
```

### Output Storage
```
data/outputs/
└── [feature]/
    ├── responses/           # Raw API responses
    ├── processed/           # Processed results
    └── evaluations/         # Quality assessments
```

### Cache Strategy
```yaml
cache_config:
  backend: "redis"  # or "file", "memory"
  ttl_seconds: 3600
  key_format: "[feature]:{hash(prompt)}"
  max_size: "1GB"
```

---

## Validation Framework

### Level 1 - Syntax & Style
```bash
ruff check src/ config/
mypy src/ --strict
python -c "import yaml; [yaml.safe_load(open(f)) for f in glob.glob('config/*.yaml')]"
```

### Level 2 - Unit Tests
```bash
pytest tests/unit/ -v --tb=short
pytest tests/unit/ --cov=src --cov-report=term-missing
```
**Expected:** >80% coverage on new code

### Level 3 - Integration Tests
```bash
# Test with real API (use test API key)
pytest tests/integration/ -v --api-key=$TEST_API_KEY

# Verify end-to-end flow
python examples/[feature]_example.py
```
**Expected:** Feature completes successfully

### Level 4 - AI-Specific Validation
```bash
# Token budget validation
python -c "from src.utils.token_counter import count; assert count(prompt) < MAX_TOKENS"

# Response quality check
python scripts/evaluate_responses.py --feature [feature] --threshold 0.8

# Cost estimation
python scripts/estimate_cost.py --feature [feature] --volume 1000
```
**Expected:**
- Token usage within budget
- Quality score >0.8
- Cost within projections

---

## AI-Specific Gotchas

### Provider-Specific
```yaml
claude:
  - "Use messages API, not completions"
  - "System prompt is separate parameter"
  - "Max 200k context window"
  - "Rate limits vary by tier"

openai:
  - "GPT-4 has different rate limits than GPT-3.5"
  - "Function calling syntax differs from Claude tools"
  - "Token counting requires tiktoken library"

general:
  - "Always implement exponential backoff"
  - "Cache identical requests to reduce costs"
  - "Log all prompts and responses for debugging"
  - "Version prompts - small changes have big impacts"
```

### Cost Control
```yaml
cost_controls:
  - "Set max_tokens to reasonable limits"
  - "Implement request budgets per user/day"
  - "Cache responses aggressively"
  - "Use cheaper models for non-critical tasks"
  - "Monitor usage dashboards daily"
```

### Quality Control
```yaml
quality_controls:
  - "Validate response structure before processing"
  - "Implement content filtering checks"
  - "Log confidence scores when available"
  - "A/B test prompt variations"
  - "Collect user feedback on outputs"
```

---

## Completion Checklist

### Technical
- [ ] All validation levels pass
- [ ] No hardcoded API keys
- [ ] Rate limiting implemented
- [ ] Caching implemented
- [ ] Error handling complete
- [ ] Logging configured

### AI Quality
- [ ] Prompts versioned in config
- [ ] Token usage within budget
- [ ] Response quality meets threshold
- [ ] Cost projections validated
- [ ] Edge cases handled

### Documentation
- [ ] Example code works
- [ ] Notebook demonstrates feature
- [ ] README updated
- [ ] Config documented

---

## Anti-Patterns for AI Projects

1. **Don't** hardcode prompts in code - use config files
2. **Don't** skip caching - API calls are expensive
3. **Don't** ignore rate limits - implement backoff
4. **Don't** trust raw responses - validate structure
5. **Don't** forget token counting - budgets matter
6. **Don't** skip logging - debugging AI is hard
7. **Don't** use production keys in dev - separate environments
8. **Don't** ignore prompt versioning - changes have big impacts
