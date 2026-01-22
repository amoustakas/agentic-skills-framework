# Orchestration Patterns Extraction

## Source
**Repository:** agentic-skills-framework (Superpowers)
**Extraction Date:** 2026-01-22

---

## 1. Complete Workflow Pipeline

The framework implements a complete software development lifecycle managed by orchestration:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        SUPERPOWERS WORKFLOW                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │ Brainstorming │ → │ Writing Plans│ → │  Execution   │              │
│  │    (Design)   │    │ (Planning)   │    │   (Build)    │              │
│  └──────────────┘    └──────────────┘    └──────────────┘              │
│         ↓                   ↓                   ↓                        │
│    Design Doc          Plan Doc          Working Code                   │
│                                                                         │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │ Code Review  │ ← │ Verification │ → │  Finishing   │              │
│  │   (Quality)  │    │  (Testing)   │    │   Branch     │              │
│  └──────────────┘    └──────────────┘    └──────────────┘              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Phase 1: Brainstorming (Design)
**Trigger:** User wants to build something
**Pattern:** Socratic design refinement

```
1. Understand project context (files, docs, commits)
2. Ask questions ONE AT A TIME
3. Prefer multiple-choice questions
4. Propose 2-3 approaches with trade-offs
5. Present design in 200-300 word sections
6. Validate each section before continuing
7. Save design to docs/plans/YYYY-MM-DD-<topic>-design.md
```

### Phase 2: Writing Plans
**Trigger:** Approved design exists
**Pattern:** Bite-sized task decomposition

```
Each task = ONE action (2-5 minutes):
- "Write the failing test"
- "Run it to make sure it fails"
- "Implement minimal code to pass"
- "Run tests and verify they pass"
- "Commit"
```

**Plan Document Structure:**
```markdown
# [Feature Name] Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use executing-plans

**Goal:** [One sentence]
**Architecture:** [2-3 sentences]
**Tech Stack:** [Key technologies]

---

### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Step 1: Write the failing test**
[Complete code]

**Step 2: Run test to verify it fails**
Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"
...
```

### Phase 3: Execution Options

**Option A: Subagent-Driven Development (Same Session)**
- Fresh subagent per task (no context pollution)
- Two-stage review after each: spec compliance, then code quality
- Faster iteration (no human-in-loop between tasks)

**Option B: Executing Plans (Parallel Session)**
- Batch execution with checkpoints
- Default: First 3 tasks
- Human review between batches

---

## 2. Subagent-Driven Development Orchestration

This is the **key orchestration pattern** for multi-agent coordination:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SUBAGENT-DRIVEN DEVELOPMENT                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Controller (Main Agent)                                                │
│  ├── Extract all tasks from plan (read ONCE)                           │
│  ├── Create TodoWrite with all tasks                                   │
│  │                                                                      │
│  └── FOR EACH TASK:                                                    │
│      │                                                                  │
│      ├── 1. Dispatch IMPLEMENTER subagent                              │
│      │   └── Provides: Full task text + context                        │
│      │   └── Subagent: Questions? → Answer → Implement → Self-review   │
│      │                                                                  │
│      ├── 2. Dispatch SPEC REVIEWER subagent                            │
│      │   └── Input: Requirements + What was built                      │
│      │   └── Output: ✅ Compliant OR ❌ Issues (missing/extra)          │
│      │   └── If issues: Implementer fixes → Re-review                  │
│      │                                                                  │
│      ├── 3. Dispatch CODE QUALITY REVIEWER subagent                    │
│      │   └── Input: Diff (BASE_SHA → HEAD_SHA)                         │
│      │   └── Output: Critical/Important/Minor issues                   │
│      │   └── If issues: Implementer fixes → Re-review                  │
│      │                                                                  │
│      └── 4. Mark task complete in TodoWrite                            │
│                                                                         │
│  AFTER ALL TASKS:                                                       │
│  └── Dispatch FINAL REVIEWER for entire implementation                 │
│  └── Use finishing-a-development-branch skill                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Key Orchestration Principles

1. **Controller provides full context** - Subagent never reads plan file
2. **Fresh subagent per task** - No context pollution between tasks
3. **Two-stage review is mandatory** - Spec compliance BEFORE code quality
4. **Review loops until approved** - Issues found → fix → re-review
5. **Questions are encouraged** - Before AND during implementation

---

## 3. Parallel Agent Dispatch Pattern

**Trigger:** 2+ independent tasks (different domains, no shared state)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    PARALLEL AGENT DISPATCH                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  IDENTIFY: Multiple independent failures/tasks                         │
│  │                                                                      │
│  ├── Group by domain:                                                  │
│  │   - File A tests: Tool approval flow                                │
│  │   - File B tests: Batch completion                                  │
│  │   - File C tests: Abort functionality                               │
│  │                                                                      │
│  DISPATCH: One agent per problem domain (parallel)                     │
│  │                                                                      │
│  │  Agent 1 ─────────────────────┐                                     │
│  │  Agent 2 ─────────────────────┼── Concurrent execution              │
│  │  Agent 3 ─────────────────────┘                                     │
│  │                                                                      │
│  INTEGRATE:                                                            │
│  ├── Read each summary                                                 │
│  ├── Verify fixes don't conflict                                       │
│  ├── Run full test suite                                               │
│  └── Integrate all changes                                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Agent Prompt Structure for Parallel Dispatch

```markdown
Fix the 3 failing tests in src/agents/agent-tool-abort.test.ts:

1. "should abort tool with partial output capture" - expects 'interrupted at'
2. "should handle mixed completed and aborted tools" - fast tool aborted
3. "should properly track pendingToolCount" - expects 3 results, gets 0

These are timing/race condition issues. Your task:

1. Read the test file and understand what each test verifies
2. Identify root cause - timing issues or actual bugs?
3. Fix by:
   - Replacing arbitrary timeouts with event-based waiting
   - Fixing bugs in abort implementation if found
   - Adjusting test expectations if testing changed behavior

Do NOT just increase timeouts - find the real issue.

Return: Summary of what you found and what you fixed.
```

---

## 4. HITL (Human-in-the-Loop) Checkpoint Patterns

### Batch Execution Checkpoints

```
Default: Execute first 3 tasks
THEN STOP and report:
  - What was implemented
  - Verification output
  - "Ready for feedback."

WAIT for human input
Apply feedback if needed
Execute next batch
Repeat until complete
```

### When to Stop and Ask

```
STOP executing immediately when:
- Hit a blocker mid-batch
- Plan has critical gaps
- Don't understand an instruction
- Verification fails repeatedly

ASK for clarification rather than guessing.
```

### Branch Completion Options

After all tasks complete, present exactly 4 options:

```
1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)
4. Discard this work
```

**For Option 4 (Discard):** Require explicit typed confirmation: `discard`

---

## 5. Review Orchestration Patterns

### Two-Stage Review Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                     REVIEW ORCHESTRATION                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STAGE 1: SPEC COMPLIANCE REVIEW                                 │
│  ├── Purpose: Verify built what was requested (nothing more/less)│
│  ├── Checks:                                                     │
│  │   - Missing requirements                                      │
│  │   - Extra/unneeded work                                       │
│  │   - Misunderstandings                                         │
│  └── Output: ✅ Spec compliant OR ❌ Issues list                  │
│                                                                  │
│  GATE: Must pass Stage 1 before Stage 2                          │
│                                                                  │
│  STAGE 2: CODE QUALITY REVIEW                                    │
│  ├── Purpose: Verify implementation is well-built                │
│  ├── Checks:                                                     │
│  │   - Clean, tested, maintainable                               │
│  │   - Architecture and design patterns                          │
│  │   - Test coverage and quality                                 │
│  └── Output: Critical/Important/Minor issues                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Code Review Issue Severity

| Level | Action |
|-------|--------|
| **Critical** | Fix immediately, blocks progress |
| **Important** | Fix before proceeding |
| **Minor** | Note for later |

---

## MCV Adaptation Notes

### For NAOS Executor Enhancement

The orchestration patterns can be adapted for multi-venture awareness:

```python
async def decompose_task(task: Task, venture_id: str) -> List[SubTask]:
    """
    NAOS decomposes tasks with venture context awareness
    """
    # Inject venture context
    context = await get_venture_context(venture_id)

    # Route to appropriate agents based on venture capabilities
    available_agents = await get_venture_agents(venture_id)

    # Decompose using SPARC methodology
    subtasks = await sparc_decompose(task, context, available_agents)

    return subtasks
```

### For Queen/Ralph Patterns

The subagent-driven-development pattern maps directly to Queen orchestration:
- Queen = Controller that extracts tasks and dispatches
- Subagents = Specialized worker agents (implementer, reviewers)
- Two-stage review = Quality gates before task completion
