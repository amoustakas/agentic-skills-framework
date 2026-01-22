# Agent Composition Patterns Extraction

## Source
**Repository:** agentic-skills-framework (Superpowers)
**Extraction Date:** 2026-01-22

---

## 1. Agent Types and Roles

The framework defines several specialized agent roles:

### Implementer Agent
**Purpose:** Execute single task with TDD discipline

```markdown
## Implementer Prompt Template

Task tool (general-purpose):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N: [task name]

    ## Task Description
    [FULL TEXT of task - paste it, don't make subagent read file]

    ## Context
    [Scene-setting: where this fits, dependencies, architectural context]

    ## Before You Begin
    If you have questions about:
    - Requirements or acceptance criteria
    - Approach or implementation strategy
    - Dependencies or assumptions
    - Anything unclear

    **Ask them now.** Raise concerns before starting.

    ## Your Job
    Once clear on requirements:
    1. Implement exactly what task specifies
    2. Write tests (following TDD if required)
    3. Verify implementation works
    4. Commit your work
    5. Self-review (see below)
    6. Report back

    ## Self-Review Checklist
    **Completeness:**
    - Did I fully implement everything in the spec?
    - Are there edge cases I didn't handle?

    **Quality:**
    - Are names clear and accurate?
    - Is code clean and maintainable?

    **Discipline:**
    - Did I avoid overbuilding (YAGNI)?
    - Did I follow existing patterns?

    **Testing:**
    - Do tests verify behavior (not just mock behavior)?
    - Are tests comprehensive?
```

### Spec Compliance Reviewer Agent
**Purpose:** Verify implementer built what was requested (nothing more, nothing less)

```markdown
## Spec Reviewer Prompt Template

Task tool (general-purpose):
  description: "Review spec compliance for Task N"
  prompt: |
    You are reviewing whether implementation matches specification.

    ## What Was Requested
    [FULL TEXT of task requirements]

    ## What Implementer Claims They Built
    [From implementer's report]

    ## CRITICAL: Do Not Trust the Report
    The implementer finished quickly. Their report may be incomplete.

    **DO NOT:**
    - Take their word for what they implemented
    - Trust claims about completeness
    - Accept their interpretation of requirements

    **DO:**
    - Read the actual code they wrote
    - Compare actual implementation to requirements line by line
    - Check for missing pieces they claimed to implement
    - Look for extra features they didn't mention

    ## Your Job
    Read implementation code and verify:

    **Missing requirements:**
    - Everything requested implemented?
    - Requirements skipped or missed?

    **Extra/unneeded work:**
    - Built things not requested?
    - Over-engineered or added unnecessary features?

    **Misunderstandings:**
    - Interpreted requirements differently?
    - Solved wrong problem?

    Report:
    - ✅ Spec compliant (if everything matches)
    - ❌ Issues found: [list with file:line references]
```

### Code Quality Reviewer Agent
**Purpose:** Verify implementation is well-built (clean, tested, maintainable)

```markdown
## Code Quality Reviewer Prompt Template

Agent: code-reviewer
Purpose: Review against plan and coding standards

1. **Plan Alignment Analysis**:
   - Compare implementation against planning document
   - Identify deviations from planned approach
   - Assess whether deviations are justified
   - Verify all planned functionality implemented

2. **Code Quality Assessment**:
   - Adherence to established patterns
   - Proper error handling, type safety
   - Code organization, naming, maintainability
   - Test coverage and quality
   - Security vulnerabilities, performance

3. **Architecture and Design Review**:
   - SOLID principles and patterns
   - Separation of concerns, loose coupling
   - Integration with existing systems
   - Scalability and extensibility

4. **Issue Classification**:
   - Critical (must fix)
   - Important (should fix)
   - Suggestions (nice to have)
```

---

## 2. Agent Communication Protocol

### Controller → Subagent Communication

**Key Principle:** Controller provides ALL context. Subagent never reads plan files.

```
┌──────────────────────────────────────────────────────────────────┐
│                  CONTEXT INJECTION PATTERN                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Controller reads plan file ONCE                                 │
│  ├── Extracts ALL tasks with full text                          │
│  ├── Notes context, dependencies, architecture                  │
│  │                                                               │
│  For each task dispatch:                                         │
│  ├── Full task text (not file path)                             │
│  ├── Scene-setting context                                       │
│  ├── Dependencies completed                                      │
│  ├── Working directory                                           │
│  └── Any answers to prior questions                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Subagent → Controller Communication

**Report Format:**
```
- What you implemented
- What you tested and results
- Files changed
- Self-review findings (if any)
- Issues or concerns
- Questions (if blocked)
```

### Handling Questions

```
IF subagent asks questions:
  1. Answer clearly and completely
  2. Provide additional context if needed
  3. Don't rush them into implementation

IF subagent is blocked:
  1. Dispatch fix subagent with specific instructions
  2. Don't try to fix manually (context pollution)
```

---

## 3. Agent Coordination Patterns

### Sequential Task Execution (Default)

```
Task 1 → Review → Task 2 → Review → Task 3 → Review → Final Review
```

**Why sequential:** Prevents conflicts, each task builds on prior

### Parallel Task Execution (When Safe)

```
Use when:
- 3+ independent tasks
- Different subsystems/files
- No shared state
- No sequential dependencies
```

**Coordination:**
```
1. Identify independent domains
2. Dispatch agents in parallel
3. Wait for all to complete
4. Review for conflicts
5. Run full test suite
6. Integrate changes
```

### Review Loops

```
┌─────────────────────────────────────────────────────────────────┐
│                      REVIEW LOOP PATTERN                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Implementer completes task                                     │
│  │                                                              │
│  Dispatch Spec Reviewer                                         │
│  ├── ✅ Compliant → Continue                                    │
│  └── ❌ Issues → Implementer fixes → Re-dispatch Spec Reviewer  │
│                                                                 │
│  Dispatch Code Quality Reviewer                                 │
│  ├── ✅ Approved → Mark task complete                           │
│  └── ❌ Issues → Implementer fixes → Re-dispatch Code Reviewer  │
│                                                                 │
│  NEVER proceed with unfixed issues                              │
│  NEVER skip the re-review after fixes                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Agent Prompt Best Practices

### Focused Prompts
```markdown
# ❌ BAD: Too broad
"Fix all the tests"

# ✅ GOOD: Specific scope
"Fix agent-tool-abort.test.ts - 3 failures related to timing"
```

### Self-Contained Prompts
```markdown
# ❌ BAD: No context
"Fix the race condition"

# ✅ GOOD: Full context
"Fix the 3 failing tests in src/agents/agent-tool-abort.test.ts:
1. 'should abort tool with partial output' - expects 'interrupted at'
2. 'should handle mixed completed and aborted' - fast tool aborted
3. 'should track pendingToolCount' - expects 3 results, gets 0"
```

### Expected Output Specified
```markdown
# ❌ BAD: Vague
"Fix it"

# ✅ GOOD: Specific
"Return: Summary of root cause and changes made"
```

### Constraints Included
```markdown
# ❌ BAD: No constraints
Agent might refactor everything

# ✅ GOOD: Clear constraints
"Do NOT change production code" or "Fix tests only"
```

---

## 5. Code Review Reception Patterns

### Response Protocol

```
WHEN receiving code review feedback:

1. READ: Complete feedback without reacting
2. UNDERSTAND: Restate requirement in own words (or ask)
3. VERIFY: Check against codebase reality
4. EVALUATE: Technically sound for THIS codebase?
5. RESPOND: Technical acknowledgment or reasoned pushback
6. IMPLEMENT: One item at a time, test each
```

### Forbidden Responses
```
NEVER:
- "You're absolutely right!"
- "Great point!"
- "Let me implement that now" (before verification)

INSTEAD:
- Restate the technical requirement
- Ask clarifying questions
- Push back with technical reasoning if wrong
- Just start working (actions > words)
```

### When to Push Back
```
Push back when:
- Suggestion breaks existing functionality
- Reviewer lacks full context
- Violates YAGNI (unused feature)
- Technically incorrect for this stack
- Legacy/compatibility reasons exist
```

---

## 6. Agent Definition Template

### YAML Frontmatter for Agents

```yaml
---
name: code-reviewer
description: |
  Use this agent when a major project step has been completed and needs review.
  Examples: After implementing authentication system, after completing API endpoints.
model: inherit  # or specify: sonnet, opus, haiku
---
```

### Agent System Prompt Structure

```markdown
You are a [Role] with expertise in [domains].

When [triggered by X], you will:

1. **[First responsibility]**:
   - Specific action
   - Specific action

2. **[Second responsibility]**:
   - Specific action
   - Specific action

Your output should be [format].
```

---

## MCV Adaptation Notes

### Agent Role Mapping

| Superpowers Agent | MCV Equivalent |
|-------------------|----------------|
| Controller | Queen Orchestrator |
| Implementer | Worker Agents (domain-specific) |
| Spec Reviewer | Compliance/Spec Agent |
| Code Reviewer | Quality Agent |
| Final Reviewer | Ralph/Critic |

### Multi-Venture Agent Isolation

```python
async def dispatch_agent(
    agent_type: str,
    task: Task,
    venture_id: str
) -> AgentResult:
    """
    Dispatch agent with venture isolation
    """
    # Get venture-specific context
    context = await get_venture_context(venture_id)

    # Inject venture boundaries
    prompt = inject_venture_boundaries(
        base_prompt=AGENT_PROMPTS[agent_type],
        venture_id=venture_id,
        allowed_resources=context.resources
    )

    # Dispatch with isolation
    return await dispatch_subagent(
        prompt=prompt,
        sandbox_config=context.sandbox
    )
```
