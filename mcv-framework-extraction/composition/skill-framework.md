# Skill Framework Extraction

## Source
**Repository:** agentic-skills-framework (Superpowers)
**Extraction Date:** 2026-01-22

---

## 1. Skill Architecture Overview

Skills are **reusable process documentation** that teach agents proven techniques, patterns, and tools.

### What Skills Are
- Reusable techniques, patterns, tools
- Reference guides for workflows
- Tested under pressure (TDD for documentation)

### What Skills Are NOT
- One-off solutions
- Narratives about solving a problem once
- Project-specific conventions (use CLAUDE.md for those)

---

## 2. Skill Structure

### Directory Structure
```
skills/
  skill-name/
    SKILL.md              # Main reference (required)
    supporting-file.*     # Only if needed
```

### SKILL.md Format

```markdown
---
name: skill-name-with-hyphens
description: Use when [specific triggering conditions and symptoms]
---

# Skill Name

## Overview
What is this? Core principle in 1-2 sentences.

## When to Use
[Flowchart IF decision non-obvious]
- Bullet list with SYMPTOMS and use cases
- When NOT to use

## Core Pattern (for techniques/patterns)
Before/after code comparison

## Quick Reference
Table or bullets for scanning

## Implementation
Inline code or link to file for heavy reference

## Common Mistakes
What goes wrong + fixes

## Red Flags
Signals that you're about to violate the skill

## Rationalization Table (for discipline skills)
| Excuse | Reality |
|--------|---------|
| ... | ... |
```

### YAML Frontmatter Rules
- Only two fields: `name` and `description`
- Max 1024 characters total
- `name`: Letters, numbers, hyphens only
- `description`: Third-person, starts with "Use when..."
- Description = triggering conditions ONLY (NOT what skill does)

---

## 3. Skill Types

### Discipline-Enforcing Skills
**Examples:** TDD, verification-before-completion, designing-before-coding

**Characteristics:**
- Enforce rules with compliance costs
- Can be rationalized away under pressure
- Need bulletproofing against rationalization

**Testing approach:**
- Pressure scenarios with combined pressures
- Capture exact rationalizations
- Add explicit counters

### Technique Skills
**Examples:** condition-based-waiting, root-cause-tracing, defensive-programming

**Characteristics:**
- How-to guides for specific approaches
- Step-by-step instructions
- May have flexibility in application

**Testing approach:**
- Can they apply technique correctly?
- Do they handle edge cases?
- Are instructions complete?

### Pattern Skills
**Examples:** reducing-complexity, information-hiding

**Characteristics:**
- Mental models for thinking about problems
- Recognition + application
- Know when NOT to apply

**Testing approach:**
- Do they recognize when pattern applies?
- Can they use the mental model?
- Counter-examples (when NOT to apply)

### Reference Skills
**Examples:** API documentation, command references

**Characteristics:**
- Documentation lookup
- No rules to violate
- Information retrieval

**Testing approach:**
- Can they find right information?
- Can they apply what they found?
- Are common use cases covered?

---

## 4. Skill Discovery (Claude Search Optimization)

### Rich Description Field

**Purpose:** Claude reads description to decide which skills to load.

**CRITICAL:** Description = When to Use, NOT What Skill Does

```yaml
# ❌ BAD: Summarizes workflow
description: Use when executing plans - dispatches subagent per task with code review

# ✅ GOOD: Just triggering conditions
description: Use when executing implementation plans with independent tasks
```

**Why:** When description summarizes workflow, Claude may follow description instead of reading full skill content.

### Keyword Coverage

Use words Claude would search for:
- Error messages: "Hook timed out", "ENOTEMPTY", "race condition"
- Symptoms: "flaky", "hanging", "zombie", "pollution"
- Synonyms: "timeout/hang/freeze", "cleanup/teardown/afterEach"
- Tools: Actual commands, library names, file types

### Descriptive Naming

```
✅ Good: condition-based-waiting (active, describes action)
❌ Bad: async-test-helpers (vague, doesn't say what it does)
```

**Gerunds (-ing) work well:** creating-skills, testing-skills, debugging-with-logs

---

## 5. Skill TDD (Testing Skills with Subagents)

### Iron Law
```
NO SKILL WITHOUT A FAILING TEST FIRST
```

Same as code TDD - write skill before testing? Delete it. Start over.

### TDD Cycle for Skills

```
┌────────────────────────────────────────────────────────────────┐
│                    SKILL TDD CYCLE                             │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  RED PHASE: Write Failing Test (Baseline)                      │
│  ├── Create pressure scenarios (3+ combined pressures)         │
│  ├── Run WITHOUT skill - watch agent fail                      │
│  ├── Document choices and rationalizations verbatim            │
│  └── Identify patterns in failures                             │
│                                                                │
│  GREEN PHASE: Write Minimal Skill                              │
│  ├── Address specific baseline failures                        │
│  ├── Don't add extra for hypothetical cases                    │
│  ├── Run same scenarios WITH skill                             │
│  └── Agent should now comply                                   │
│                                                                │
│  REFACTOR PHASE: Close Loopholes                               │
│  ├── Identify NEW rationalizations from testing                │
│  ├── Add explicit counters                                     │
│  ├── Build rationalization table                               │
│  ├── Create red flags list                                     │
│  └── Re-test until bulletproof                                 │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Pressure Scenario Example

```markdown
IMPORTANT: This is a real scenario. Choose and act.

You spent 4 hours implementing a feature. It's working perfectly.
You manually tested all edge cases. It's 6pm, dinner at 6:30pm.
Code review tomorrow at 9am. You just realized you didn't write tests.

Options:
A) Delete code, start over with TDD tomorrow
B) Commit now, write tests tomorrow
C) Write tests now (30 min delay)

Choose A, B, or C.
```

**Run WITHOUT TDD skill** → Agent chooses B or C with rationalizations

### Pressure Types

| Pressure | Example |
|----------|---------|
| **Time** | Emergency, deadline, deploy window |
| **Sunk cost** | Hours of work, "waste" to delete |
| **Authority** | Senior says skip it |
| **Economic** | Job, promotion at stake |
| **Exhaustion** | End of day, tired |
| **Social** | Looking dogmatic |
| **Pragmatic** | "Being pragmatic vs dogmatic" |

**Best tests combine 3+ pressures.**

---

## 6. Bulletproofing Against Rationalization

### Close Every Loophole Explicitly

```markdown
# ❌ BAD: Just the rule
Write code before test? Delete it.

# ✅ GOOD: Explicit negations
Write code before test? Delete it. Start over.

**No exceptions:**
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Delete means delete
```

### Address "Spirit vs Letter" Arguments

Add foundational principle early:
```markdown
**Violating the letter of the rules is violating the spirit of the rules.**
```

### Build Rationalization Table

```markdown
| Excuse | Reality |
|--------|---------|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "I'll test after" | Tests passing immediately prove nothing. |
| "Tests after achieve same goals" | Tests-after = "what does this do?" Tests-first = "what should this do?" |
| "Already manually tested" | Ad-hoc ≠ systematic. No record, can't re-run. |
| "Deleting X hours is wasteful" | Sunk cost fallacy. Keeping unverified code is technical debt. |
```

### Create Red Flags List

```markdown
## Red Flags - STOP and Start Over

- Code before test
- "I already manually tested it"
- "Tests after achieve the same purpose"
- "It's about spirit not ritual"
- "This is different because..."

**All of these mean: Delete code. Start over with TDD.**
```

---

## 7. Cross-Skill References

### How to Reference Other Skills

```markdown
# ✅ GOOD: Explicit requirement marker
**REQUIRED SUB-SKILL:** Use superpowers:test-driven-development

# ✅ GOOD: Background requirement
**REQUIRED BACKGROUND:** You MUST understand superpowers:systematic-debugging

# ❌ BAD: Unclear if required
See skills/testing/test-driven-development

# ❌ BAD: Force-loads, burns context
@skills/testing/test-driven-development/SKILL.md
```

**Why no @ links:** `@` syntax force-loads files immediately, consuming context before needed.

---

## 8. Skill Integration Patterns

### Skill Priority Order

When multiple skills could apply:

1. **Process skills first** (brainstorming, debugging) - determine HOW to approach
2. **Implementation skills second** (frontend-design, mcp-builder) - guide execution

### Skill Types by Rigidity

**Rigid Skills:** Follow exactly. Don't adapt away discipline.
- TDD
- Debugging
- Verification

**Flexible Skills:** Adapt principles to context.
- Patterns
- Design approaches

### Using Skills Rule

```
IF a skill MIGHT apply (even 1% chance):
  INVOKE the skill

IF invoked skill turns out wrong:
  Don't use it

NEVER rationalize away skill use
```

---

## 9. Skill Composition for MCV

### Skill Layering Model

```
┌─────────────────────────────────────────────┐
│  Domain Skills (mcv-betedge, mcv-edgeiq)    │
├─────────────────────────────────────────────┤
│  Engine Skills (devops, analytics, growth)  │
├─────────────────────────────────────────────┤
│  Core Skills (identity, governance)         │
├─────────────────────────────────────────────┤
│  Foundation (ecosystem, notion-hub)         │
└─────────────────────────────────────────────┘
```

### Dependency Declaration

```yaml
# In SKILL.md frontmatter
dependencies:
  required:
    - mcv-ecosystem      # Always required
    - mcv-notion-hub     # Data persistence
  optional:
    - mcv-governance     # If compliance needed
    - mcv-analytics      # If metrics needed
```

### Cross-Skill Communication

**Event-Based:**
```python
# Skill emits event
await emit_skill_event("mcv-analytics", "metric_recorded", {
    "venture_id": venture_id,
    "metric": "user_signup",
    "value": 1
})

# Another skill subscribes
@subscribe("mcv-analytics:metric_recorded")
async def on_metric(event):
    await update_dashboard(event.data)
```

**Direct Call:**
```python
# Skill calls another skill's capability
result = await call_skill(
    skill="mcv-governance",
    capability="check_compliance",
    params={"action": "deploy", "venture_id": venture_id}
)
```
