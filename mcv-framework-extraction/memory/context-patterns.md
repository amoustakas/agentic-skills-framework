# Memory and Context Patterns Extraction

## Source
**Repository:** agentic-skills-framework (Superpowers)
**Extraction Date:** 2026-01-22

---

## 1. Context Injection Patterns

### Controller-to-Subagent Context Passing

The framework's key pattern: **Controller provides ALL context. Subagent never reads files.**

```
┌────────────────────────────────────────────────────────────────┐
│                 CONTEXT INJECTION FLOW                         │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Controller                                                    │
│  │                                                             │
│  ├── Read plan file ONCE                                       │
│  │   └── Extract ALL tasks with full text                      │
│  │   └── Note context, dependencies, architecture              │
│  │                                                             │
│  └── For each subagent dispatch, inject:                       │
│      ├── Full task text (NOT file path)                        │
│      ├── Scene-setting context                                 │
│      ├── Dependencies completed                                │
│      ├── Working directory                                     │
│      └── Answers to prior questions                            │
│                                                                │
│  Subagent                                                      │
│  │                                                             │
│  └── Receives complete context in prompt                       │
│      └── NEVER reads plan file                                 │
│      └── NEVER needs file system access for task info          │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Why This Pattern Works

**Benefits:**
- No file reading overhead for subagent
- Controller curates exactly what context is needed
- Subagent gets complete information upfront
- Questions surfaced before work begins (not after)
- Fresh subagent per task = no context pollution

**Anti-pattern:**
```
❌ "Go read docs/plans/feature-plan.md and implement Task 3"
✅ "Here is Task 3: [FULL TEXT]. Context: [DETAILS]."
```

---

## 2. Progressive Skill Loading

### Skill Discovery Architecture

Skills use a two-phase loading model to preserve context:

```
PHASE 1: METADATA LOADING (Startup)
├── Load name + description from ALL skills
├── Claude uses descriptions to decide which to invoke
└── Very low token cost

PHASE 2: ON-DEMAND LOADING (When Triggered)
├── Load SKILL.md body when skill becomes relevant
├── Load additional files only as needed
└── Preserves context for actual conversation
```

### Token Efficiency Strategies

**Keep frequently-loaded skills small:**
```
getting-started workflows: <150 words each
Frequently-loaded skills: <200 words total
Other skills: <500 words
```

**Move details to separate files:**
```markdown
# ❌ BAD: Everything in SKILL.md
[600 lines of API reference]

# ✅ GOOD: Reference files
## Advanced features
See [REFERENCE.md](REFERENCE.md) for API details
```

**Cross-reference instead of repeat:**
```markdown
# ❌ BAD: Repeat workflow
[20 lines repeated instructions]

# ✅ GOOD: Reference
**REQUIRED:** Use [other-skill-name] for workflow.
```

---

## 3. Skill File Organization Patterns

### Keep References One Level Deep

Claude may partially read nested references. All reference files should link directly from SKILL.md.

```markdown
# ❌ BAD: Too deep (nested references)
SKILL.md → advanced.md → details.md

# ✅ GOOD: One level
SKILL.md
├── [Advanced features](advanced.md)
├── [API reference](reference.md)
└── [Examples](examples.md)
```

### Domain-Specific Organization

For skills with multiple domains, organize to avoid loading irrelevant context:

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns)
```

When user asks about sales, Claude reads only `reference/sales.md`.

### Table of Contents for Long Files

For reference files >100 lines, include TOC at top:

```markdown
# API Reference

## Contents
- Authentication and setup
- Core methods (create, read, update, delete)
- Advanced features
- Error handling
- Code examples

## Authentication
...
```

---

## 4. Context Window Management

### The Context Window is a Public Good

Skill content shares context with:
- System prompt
- Conversation history
- Other skills' metadata
- User's request

**Default assumption:** Claude is already very smart. Only add context Claude doesn't already have.

### Challenge Each Addition

Ask:
- "Does Claude really need this explanation?"
- "Can I assume Claude knows this?"
- "Does this paragraph justify its token cost?"

### Good vs Bad Example

```markdown
# ❌ BAD: Too verbose (~150 tokens)
PDF (Portable Document Format) files are a common file format that
contains text, images, and other content. To extract text from a PDF,
you'll need to use a library. There are many libraries available...

# ✅ GOOD: Concise (~50 tokens)
## Extract PDF text
Use pdfplumber for text extraction:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
```

---

## 5. Git Worktrees for Context Isolation

### Pattern: Isolated Workspaces

Git worktrees create isolated workspaces sharing same repository:

```bash
# Create worktree with new branch
git worktree add .worktrees/feature-auth -b feature/auth
cd .worktrees/feature-auth

# Run project setup
npm install  # or cargo build, pip install, etc.

# Verify clean baseline
npm test  # All tests should pass

# Work in isolation
# Changes don't affect main workspace
```

### Directory Selection Priority

```
1. Check existing directories:
   .worktrees/  (preferred, hidden)
   worktrees/   (alternative)

2. Check CLAUDE.md for preference

3. Ask user:
   1. .worktrees/ (project-local, hidden)
   2. ~/.config/superpowers/worktrees/<project>/
```

### Safety Verification

Before creating project-local worktree:
```bash
# MUST verify directory is git-ignored
git check-ignore -q .worktrees 2>/dev/null

# If NOT ignored: Add to .gitignore and commit
```

---

## 6. Cross-Agent Communication Patterns

### TodoWrite for Progress Tracking

```
Controller creates TodoWrite with all tasks
├── Mark in_progress when starting task
├── Mark completed when task done
└── Provides visibility to human

Subagent follows same pattern
├── Can update progress
├── Creates sub-todos if needed
```

### Question Protocol

```
IF subagent has questions BEFORE starting:
  Ask them now
  Controller answers
  Subagent proceeds

IF subagent blocked DURING work:
  Stop and ask
  Don't guess or make assumptions
```

### Report Format

Subagent reports back:
```
- What you implemented
- What you tested and results
- Files changed
- Self-review findings (if any)
- Issues or concerns
```

---

## 7. Memory Patterns for MCV

### Venture-Scoped Memory

```sql
-- Memory table with venture isolation
CREATE TABLE agent_memories (
    id UUID PRIMARY KEY,
    venture_id UUID NOT NULL REFERENCES ventures(id),
    agent_id TEXT NOT NULL,
    memory_type TEXT CHECK (memory_type IN ('episodic', 'semantic', 'pattern')),
    content TEXT NOT NULL,
    embedding VECTOR(1536),
    confidence FLOAT DEFAULT 1.0,
    created_at TIMESTAMPTZ DEFAULT now()
);

-- RLS for venture isolation
ALTER TABLE agent_memories ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Venture memory isolation" ON agent_memories
    USING (venture_id = current_setting('app.current_venture_id')::uuid);
```

### Memory Types

| Type | Purpose | Example |
|------|---------|---------|
| **Episodic** | Short-term session context | Current task state, recent interactions |
| **Semantic** | Long-term knowledge | User preferences, project patterns |
| **Pattern** | Distilled reasoning | What worked, what didn't |

### Context Injection for Multi-Venture

```python
async def inject_venture_context(
    base_prompt: str,
    venture_id: str
) -> str:
    """
    Inject venture-specific context into agent prompt
    """
    # Get venture config
    config = await get_venture_config(venture_id)

    # Get relevant memories
    memories = await get_relevant_memories(
        venture_id=venture_id,
        memory_types=['semantic', 'pattern'],
        limit=10
    )

    # Build context block
    context = f"""
## Venture Context
- Venture: {config.name}
- Domain: {config.domain}
- Constraints: {config.constraints}

## Relevant Knowledge
{format_memories(memories)}
"""

    return base_prompt.replace("{VENTURE_CONTEXT}", context)
```

---

## 8. Key Principles Summary

### Context Efficiency

1. **Controller provides context** - Subagent never reads plan files
2. **Progressive loading** - Only load what's needed when needed
3. **One level deep** - Avoid nested file references
4. **Challenge every token** - Does this justify context cost?

### Context Isolation

1. **Fresh subagent per task** - No context pollution
2. **Git worktrees** - Physical workspace isolation
3. **Venture scoping** - Memory isolated per venture

### Context Flow

1. **Extract once, inject many** - Controller extracts, subagents receive
2. **Questions before work** - Surface unclear context early
3. **Report format** - Structured context back to controller
