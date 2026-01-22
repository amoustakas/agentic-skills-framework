# MCV Framework Extraction Summary

## Source Repository
**Repository:** agentic-skills-framework (Superpowers by @obra)
**Extraction Date:** 2026-01-22
**Purpose:** Extract patterns, frameworks, and skill structures for MCV NAOS integration

---

## What This Repository Provides

The agentic-skills-framework is a **complete software development workflow system** built on composable "skills" that teach AI agents proven development practices. It implements:

1. **Complete Workflow Pipeline** - From brainstorming through completion
2. **Subagent Orchestration** - Multi-agent coordination with quality gates
3. **Skill Architecture** - How to write, test, and compose reusable skills
4. **Quality Gates** - TDD, verification, systematic debugging
5. **Context Management** - Efficient context passing between agents

---

## Extraction Contents

### `/orchestration/orchestration-patterns.md`
- **Workflow Pipeline:** Brainstorming → Planning → Execution → Verification → Completion
- **Subagent-Driven Development:** Fresh agent per task + two-stage review
- **Parallel Agent Dispatch:** When and how to parallelize independent tasks
- **HITL Checkpoint Patterns:** Batch execution with human review points
- **Branch Completion Options:** Merge/PR/Keep/Discard workflow

### `/agents/agent-composition-patterns.md`
- **Agent Types:** Implementer, Spec Reviewer, Code Quality Reviewer
- **Communication Protocol:** Controller-to-subagent context injection
- **Coordination Patterns:** Sequential vs parallel, review loops
- **Prompt Best Practices:** Focused, self-contained, output-specified
- **Code Review Reception:** How agents should handle feedback

### `/composition/skill-framework.md`
- **Skill Architecture:** SKILL.md structure, YAML frontmatter
- **Skill Types:** Discipline-enforcing, Technique, Pattern, Reference
- **Skill TDD:** Testing skills with pressure scenarios
- **Bulletproofing:** Rationalization tables, red flags, explicit negations
- **Cross-Skill References:** How to reference other skills
- **Skill Composition:** Layering, dependencies, communication

### `/patterns/quality-gates.md`
- **TDD Iron Law:** No production code without failing test first
- **Red-Green-Refactor:** Complete cycle with verification
- **Verification Before Completion:** Evidence before claims
- **Systematic Debugging:** Four-phase root cause process
- **Testing Anti-Patterns:** What to avoid in tests
- **Condition-Based Waiting:** Replace arbitrary timeouts
- **Code Review Gates:** Two-stage review (spec then quality)

### `/memory/context-patterns.md`
- **Context Injection:** Controller provides ALL context
- **Progressive Skill Loading:** Metadata first, content on-demand
- **Token Efficiency:** Keep skills small, use references
- **Git Worktrees:** Workspace isolation pattern
- **Cross-Agent Communication:** TodoWrite, question protocol
- **Venture-Scoped Memory:** MCV memory isolation pattern

---

## Key Value Extractions

### 1. Subagent-Driven Development Pattern
The most valuable orchestration pattern. Maps directly to Queen/Ralph:

```
Controller (Queen) reads plan ONCE
  └── For each task:
      ├── Dispatch Implementer with full context
      ├── Dispatch Spec Reviewer (must pass)
      ├── Dispatch Code Quality Reviewer (must pass)
      └── Mark complete
```

### 2. Two-Stage Review Gate
Critical quality control pattern:

```
Stage 1: SPEC COMPLIANCE
  └── Did they build what was requested?
      ├── Nothing missing?
      └── Nothing extra?

GATE: Must pass Stage 1 before Stage 2

Stage 2: CODE QUALITY
  └── Is it well-built?
      ├── Clean, tested, maintainable?
      └── Critical/Important/Minor issues
```

### 3. Context Injection Pattern
Key efficiency pattern for multi-agent systems:

```
Controller extracts ALL tasks from plan (read ONCE)
For each subagent dispatch:
  ├── Full task text (NOT file path)
  ├── Scene-setting context
  ├── Dependencies completed
  └── Prior answers/decisions

Subagent NEVER reads plan file
```

### 4. Skill TDD
Testing methodology for process documentation:

```
RED: Run scenario WITHOUT skill → watch agent fail
GREEN: Write skill → watch agent comply
REFACTOR: Close loopholes → add explicit counters
```

### 5. Rationalization Prevention
Bulletproofing discipline skills:

```
| Excuse | Reality |
|--------|---------|
| "Just this once" | No exceptions |
| "Spirit not letter" | Violating letter IS violating spirit |
| "Tests after same goals" | Tests-after = "what does it do?" Tests-first = "what should it do?" |
```

---

## MCV Integration Recommendations

### For NAOS Executor
- Adopt subagent-driven-development orchestration pattern
- Implement two-stage review gates
- Use context injection (controller provides context)

### For Queen/Ralph
- Map Controller → Queen
- Map Reviewers → Ralph/Critic role
- Use parallel dispatch for independent ventures

### For mcv-hive-mind
- Adopt progressive loading pattern
- Implement venture-scoped memory isolation
- Use episodic/semantic/pattern memory types

### For All Skills
- Adopt skill TDD methodology
- Use bulletproofing patterns for discipline skills
- Follow context efficiency principles

---

## Extracted Files

```
mcv-framework-extraction/
├── EXTRACTION-SUMMARY.md           # This file
├── orchestration/
│   └── orchestration-patterns.md   # Workflow and agent dispatch
├── agents/
│   └── agent-composition-patterns.md # Agent types and communication
├── composition/
│   └── skill-framework.md          # Skill architecture and TDD
├── patterns/
│   └── quality-gates.md            # TDD, verification, debugging
└── memory/
    └── context-patterns.md         # Context injection and management
```

---

## Philosophy Alignment

The framework's philosophy aligns well with MCV goals:

| Superpowers Principle | MCV Application |
|----------------------|-----------------|
| Test-Driven Development | Quality-first development |
| Systematic over ad-hoc | Process over guessing |
| Complexity reduction | Simplicity as primary goal |
| Evidence over claims | Verify before declaring success |
| YAGNI ruthlessly | Build only what's needed |
| Violating letter = violating spirit | No rationalization allowed |

---

## Next Steps

1. **Review extracted patterns** against MCV architecture needs
2. **Select patterns** for immediate integration
3. **Adapt terminology** (Controller → Queen, etc.)
4. **Implement** in priority order:
   - Subagent-driven orchestration (P0)
   - Two-stage review gates (P0)
   - Context injection (P1)
   - Skill TDD methodology (P1)

---

*Extracted from agentic-skills-framework by Claude for MCV NAOS Skills Expansion*
