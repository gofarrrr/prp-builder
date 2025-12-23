# Proactive Guidance & Pushback Framework

How to guide users toward better PRPs and push back when choices are inappropriate.

---

## Core Philosophy

### The Genius Intern Framework
AI agents are like brilliant interns—they need context about quality expectations in YOUR specific situation. PRPs must provide:
- What "good" looks like in this codebase
- Decision frameworks for ambiguous situations
- Examples with reasoning, not just examples

### Async Remote Teammate Approach
Write PRPs the way you'd write for a smart remote colleague:
- **Clear** - No ambiguity about what to do
- **Comprehensive** - All context included, no assumptions
- **Decisively opinionated** - Make choices, don't hedge

---

## Proactive Guidance Triggers

### During Scope Discovery

**Trigger:** User describes scope as "small" but mentions multiple systems
```
Guidance: "You mentioned [system A] and [system B]. Features touching multiple
systems typically benefit from a Story or Base PRP rather than Task PRP.
The extra structure helps prevent missed integration points.

Would you like to:
1. Proceed with Task PRP (if changes are truly isolated)
2. Upgrade to Story PRP (recommended for multi-system work)"
```

**Trigger:** User wants Planning PRP for something straightforward
```
Guidance: "Planning PRPs are designed for features requiring research before
specification—like evaluating new technologies or unclear requirements.

Your feature [X] seems well-defined. A Base PRP would let you start
implementation sooner while still being comprehensive.

What's driving the need for research phase?"
```

**Trigger:** User wants Task PRP for something complex
```
Guidance: "Task PRPs work best for atomic changes—single file fixes, small
enhancements. Your feature involves [complexity indicators].

Under-specifying complex work leads to:
- Missed edge cases
- Inconsistent patterns
- Rework when integration fails

Recommend: Story PRP with clear task breakdown. Takes 10 more minutes to
create, saves hours of back-and-forth."
```

---

### During Context Gathering

**Trigger:** User provides vague file references
```
Guidance: "I notice references like 'the auth module' and 'user service'.
Vague references cost tokens—agents spend context discovering what you
already know.

Can you provide specific paths? For example:
- 'src/auth/jwt_handler.py' instead of 'the auth module'
- 'src/services/user_service.py:45-67' for pattern references

If you're unsure, I can search the codebase to find exact locations."
```

**Trigger:** User skips acceptance criteria
```
Guidance: "You haven't defined acceptance criteria yet. Without testable
success conditions:
- Agents can't self-validate completion
- You'll need to manually verify everything
- Scope creep becomes invisible

What must be true when this is done? Think:
'I can verify this by running [command] and seeing [result]'"
```

**Trigger:** User provides too much context
```
Guidance: "You've provided extensive documentation links. Context overload
is as problematic as context starvation:
- Agents have finite attention budgets
- Irrelevant info competes with relevant info
- More tokens ≠ better results

Let's filter to only what's needed for THIS implementation:
- Which of these docs directly inform coding decisions?
- Can we link to specific sections instead of whole documents?"
```

---

### During Template Selection

**Trigger:** User selects Generative AI template for non-AI work
```
Guidance: "The Generative AI template is specialized for LLM integrations—
it includes sections for prompt versioning, token budgets, and provider
configs that don't apply here.

For [your feature type], I recommend [appropriate template] because
[specific reasons]."
```

**Trigger:** User skips Planning phase for unclear requirements
```
Guidance: "I'm noticing several open questions in your description:
- [Question 1]
- [Question 2]

Building a PRP with unclear requirements leads to:
- Assumptions that may be wrong
- Rework when clarified
- Scope creep mid-implementation

Options:
1. Answer these questions now, then proceed with [template]
2. Start with Planning PRP to research and clarify first
3. Proceed anyway (not recommended—may need to restart)"
```

---

## Pushback Patterns

### Firm Pushback (Redirect Required)

Use when the approach will likely fail:

**Pattern: Over-Engineering Detection**
```
User wants: Exhaustive edge case handling for simple CRUD
Pushback: "You've listed 15 edge cases for a basic create endpoint.
This level of specification:
- Suggests scope creep or unclear requirements
- Will consume tokens on unlikely scenarios
- May indicate the feature should be split

Let's focus on the 3-4 most likely cases. We can add others when
they actually occur. Which are most critical?"
```

**Pattern: Missing Foundation**
```
User wants: Jump to implementation without understanding codebase
Pushback: "You want to implement [feature] but haven't identified:
- Existing patterns to follow
- Integration points
- Testing conventions

PRPs built without codebase analysis often conflict with existing
architecture. Recommend: Let me analyze first, then we build the PRP
with accurate references."
```

**Pattern: Wrong Template Insistence**
```
User insists: Task PRP for multi-week feature
Pushback: "I understand you want to keep it simple, but Task PRPs for
complex features create problems:
- Missing sections means missing context
- Agents work piecemeal without seeing the whole
- Validation gaps let errors propagate

I can create a streamlined Base PRP—same concepts, appropriate depth.
This isn't about bureaucracy; it's about giving agents what they need
to succeed."
```

---

### Soft Pushback (Suggestion with Rationale)

Use when improvement is possible but current approach may work:

**Pattern: Enhancement Suggestion**
```
"Your PRP covers the basics. Consider adding:
- [Suggestion 1]: Because [specific benefit]
- [Suggestion 2]: Because [specific benefit]

These aren't required, but they'd reduce agent questions and improve
first-attempt success rate. Add them?"
```

**Pattern: Pattern Reference**
```
"You've described the implementation, but I found an existing pattern
at [file:line] that does something similar.

Referencing it would:
- Ensure consistency with codebase conventions
- Reduce agent decision-making burden
- Speed up implementation

Want me to add the reference?"
```

**Pattern: Validation Gap**
```
"Tasks 2 and 3 don't have validation commands. Adding them would:
- Catch errors before they compound
- Provide clear completion signals
- Enable faster iteration

For Task 2, something like: `pytest tests/[specific].py -v`
For Task 3: `ruff check [path] && mypy [path]`

Should I add these?"
```

---

## Educational Moments

### Explain the "Why" Behind Recommendations

**When suggesting structure:**
```
"I'm recommending [structure] because:

Context engineering principle: [specific principle]
- Your situation: [how it applies]
- Risk if ignored: [what goes wrong]
- Benefit of following: [what improves]

This isn't arbitrary—it's based on how agents process information."
```

**When correcting anti-patterns:**
```
"[Current approach] is an anti-pattern because:

What happens: [failure mode]
Why it happens: [underlying reason]
Better approach: [recommendation]
Example: [concrete illustration]

Would you like me to restructure?"
```

---

## Response Templates

### For Scope Mismatches
```
📊 **Scope Check**

Your description suggests [complexity level], but you selected [template type].

| Indicator | Your Feature | Template Expectation |
|-----------|--------------|---------------------|
| Files affected | [N] | [expected range] |
| Systems touched | [list] | [expected] |
| Dependencies | [complexity] | [expected] |

**Recommendation:** [template] would provide [specific benefits].

Proceed with current selection, or switch?
```

### For Missing Context
```
⚠️ **Context Gap Detected**

Your PRP is missing [element type]:
- **Impact:** [what goes wrong without it]
- **Fix:** [how to provide it]

Example of what's needed:
```
[concrete example]
```

Can you provide this, or should I help discover it?
```

### For Over-Specification
```
✂️ **Simplification Opportunity**

Section [X] has more detail than agents typically need:
- Current: [what's there]
- Recommended: [what's sufficient]
- Why: [reasoning]

Agents work best with high-signal, concise context. Extra detail
competes for attention without adding value.

Trim it down?
```

---

## Checklist: Before Finalizing Any PRP

Run these checks and surface issues proactively:

### Scope Alignment
- [ ] Template matches actual complexity
- [ ] No over-engineering for simple tasks
- [ ] No under-specification for complex tasks

### Context Quality
- [ ] All references are specific (file:line, not "the module")
- [ ] No vague descriptions where specifics exist
- [ ] Context budget is appropriate (not overloaded)

### Completeness
- [ ] Acceptance criteria are testable
- [ ] Each task has validation
- [ ] Dependencies are explicit
- [ ] Gotchas are documented

### Agent-Readiness
- [ ] Another dev could implement from PRP alone
- [ ] No assumptions requiring clarification
- [ ] Patterns referenced, not described
- [ ] Decisive opinions, not hedged options
