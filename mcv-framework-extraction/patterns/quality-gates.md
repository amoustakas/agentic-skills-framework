# Quality Gates Patterns Extraction

## Source
**Repository:** agentic-skills-framework (Superpowers)
**Extraction Date:** 2026-01-22

---

## 1. Test-Driven Development (TDD)

### The Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

Write code before test? **Delete it. Start over.**

### Red-Green-Refactor Cycle

```
┌────────────────────────────────────────────────────────────────┐
│                    RED-GREEN-REFACTOR                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌────────┐         ┌────────┐         ┌────────┐             │
│  │  RED   │ ─────→ │ GREEN  │ ─────→ │REFACTOR│             │
│  │ Write  │         │Minimal │         │ Clean  │             │
│  │failing │         │ code   │         │  up    │             │
│  │ test   │         │to pass │         │        │             │
│  └────────┘         └────────┘         └────────┘             │
│       │                                      │                 │
│       └──────────────────────────────────────┘                 │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### RED - Write Failing Test

**Requirements:**
- One behavior
- Clear name
- Real code (no mocks unless unavoidable)

**Good Test:**
```typescript
test('retries failed operations 3 times', async () => {
  let attempts = 0;
  const operation = () => {
    attempts++;
    if (attempts < 3) throw new Error('fail');
    return 'success';
  };

  const result = await retryOperation(operation);

  expect(result).toBe('success');
  expect(attempts).toBe(3);
});
```

### Verify RED - Watch It Fail

**MANDATORY. Never skip.**

Confirm:
- Test fails (not errors)
- Failure message is expected
- Fails because feature missing (not typos)

**Test passes?** You're testing existing behavior. Fix test.

### GREEN - Minimal Code

Write **simplest code** to pass the test.

```typescript
// ✅ GOOD: Just enough to pass
async function retryOperation<T>(fn: () => Promise<T>): Promise<T> {
  for (let i = 0; i < 3; i++) {
    try {
      return await fn();
    } catch (e) {
      if (i === 2) throw e;
    }
  }
  throw new Error('unreachable');
}

// ❌ BAD: Over-engineered
async function retryOperation<T>(
  fn: () => Promise<T>,
  options?: {
    maxRetries?: number;
    backoff?: 'linear' | 'exponential';
    onRetry?: (attempt: number) => void;
  }
): Promise<T> {
  // YAGNI
}
```

### REFACTOR - Clean Up

After green only:
- Remove duplication
- Improve names
- Extract helpers

**Keep tests green. Don't add behavior.**

---

## 2. Verification Before Completion

### The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command **in this message**, you cannot claim it passes.

### The Gate Function

```
BEFORE claiming any status or satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

### Common Claim Requirements

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Test output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing |
| Bug fixed | Original symptom passes | Code changed |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |

### Red Flags - STOP

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification
- About to commit/push/PR without verification
- Trusting agent success reports
- Thinking "just this once"

### Key Patterns

**Tests:**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**Regression tests (TDD Red-Green):**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**Agent delegation:**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

---

## 3. Systematic Debugging

### The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes.

### The Four Phases

```
┌────────────────────────────────────────────────────────────────┐
│                    SYSTEMATIC DEBUGGING                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  PHASE 1: ROOT CAUSE INVESTIGATION                             │
│  ├── Read error messages carefully (complete stack traces)     │
│  ├── Reproduce consistently                                    │
│  ├── Check recent changes (git diff, commits)                  │
│  ├── Gather evidence in multi-component systems                │
│  └── Trace data flow                                           │
│                                                                │
│  PHASE 2: PATTERN ANALYSIS                                     │
│  ├── Find working examples in codebase                         │
│  ├── Compare against references                                │
│  ├── Identify differences                                      │
│  └── Understand dependencies                                   │
│                                                                │
│  PHASE 3: HYPOTHESIS AND TESTING                               │
│  ├── Form single hypothesis                                    │
│  ├── Test minimally (ONE variable)                             │
│  ├── Verify before continuing                                  │
│  └── If wrong, form NEW hypothesis                             │
│                                                                │
│  PHASE 4: IMPLEMENTATION                                       │
│  ├── Create failing test case                                  │
│  ├── Implement single fix                                      │
│  ├── Verify fix                                                │
│  └── If 3+ fixes failed: QUESTION ARCHITECTURE                 │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 3+ Fixes Failed Rule

```
IF < 3 failed fixes:
  Return to Phase 1, re-analyze

IF ≥ 3 failed fixes:
  STOP and question the architecture:
  - Is this pattern fundamentally sound?
  - Are we "sticking with it through sheer inertia"?
  - Should we refactor architecture vs. continue fixing symptoms?

  Discuss with human partner before more fixes
```

### Root Cause Tracing

When bugs manifest deep in call stack:

```
1. Observe the symptom
   Error: git init failed in /Users/jesse/project/packages/core

2. Find immediate cause
   await execFileAsync('git', ['init'], { cwd: projectDir });

3. Ask: What called this?
   WorktreeManager.createSessionWorktree(projectDir, sessionId)
   → called by Session.initializeWorkspace()
   → called by Session.create()
   → called by test at Project.create()

4. Keep tracing up
   projectDir = '' (empty string!)
   Empty string as cwd resolves to process.cwd()

5. Find original trigger
   const context = setupCoreTest(); // Returns { tempDir: '' }
   Project.create('name', context.tempDir); // Accessed before beforeEach!
```

**NEVER fix just where the error appears. Trace back to original trigger.**

### Defense-in-Depth Validation

After finding root cause, validate at EVERY layer:

```
Layer 1: Entry Point Validation
├── Reject obviously invalid input at API boundary

Layer 2: Business Logic Validation
├── Ensure data makes sense for this operation

Layer 3: Environment Guards
├── Prevent dangerous operations in specific contexts

Layer 4: Debug Instrumentation
├── Capture context for forensics
```

**All four layers are necessary.** Different layers catch different cases.

---

## 4. Testing Anti-Patterns to Avoid

### The Iron Laws

```
1. NEVER test mock behavior
2. NEVER add test-only methods to production classes
3. NEVER mock without understanding dependencies
```

### Anti-Pattern 1: Testing Mock Behavior

```typescript
// ❌ BAD: Testing that the mock exists
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});

// ✅ GOOD: Test real component
test('renders sidebar', () => {
  render(<Page />);  // Don't mock sidebar
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});
```

### Anti-Pattern 2: Test-Only Methods

```typescript
// ❌ BAD: destroy() only used in tests
class Session {
  async destroy() {  // Pollutes production code
    await this._workspaceManager?.destroyWorkspace(this.id);
  }
}

// ✅ GOOD: Test utilities handle cleanup
// In test-utils/
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}
```

### Anti-Pattern 3: Mocking Without Understanding

```typescript
// ❌ BAD: Mock breaks test logic
test('detects duplicate server', () => {
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn() // Breaks side effect test depends on
  }));
});

// ✅ GOOD: Mock at correct level
test('detects duplicate server', () => {
  vi.mock('MCPServerManager'); // Just mock slow server startup
});
```

### Gate Function Before Mocking

```
BEFORE mocking any method:
  1. What side effects does real method have?
  2. Does this test depend on any of those effects?
  3. Do I fully understand what test needs?

IF depends on side effects:
  Mock at lower level
  NOT the high-level method test depends on
```

---

## 5. Condition-Based Waiting (vs Arbitrary Timeouts)

### Core Principle

```
Wait for the ACTUAL CONDITION you care about,
not a guess about how long it takes.
```

### Pattern

```typescript
// ❌ BEFORE: Guessing at timing
await new Promise(r => setTimeout(r, 50));
const result = getResult();
expect(result).toBeDefined();

// ✅ AFTER: Waiting for condition
await waitFor(() => getResult() !== undefined);
const result = getResult();
expect(result).toBeDefined();
```

### Implementation

```typescript
async function waitFor<T>(
  condition: () => T | undefined | null | false,
  description: string,
  timeoutMs = 5000
): Promise<T> {
  const startTime = Date.now();

  while (true) {
    const result = condition();
    if (result) return result;

    if (Date.now() - startTime > timeoutMs) {
      throw new Error(`Timeout waiting for ${description} after ${timeoutMs}ms`);
    }

    await new Promise(r => setTimeout(r, 10)); // Poll every 10ms
  }
}
```

### Quick Reference

| Scenario | Pattern |
|----------|---------|
| Wait for event | `waitFor(() => events.find(e => e.type === 'DONE'))` |
| Wait for state | `waitFor(() => machine.state === 'ready')` |
| Wait for count | `waitFor(() => items.length >= 5)` |
| Wait for file | `waitFor(() => fs.existsSync(path))` |

---

## 6. Code Review Quality Gates

### Two-Stage Review

```
STAGE 1: SPEC COMPLIANCE
├── Built what was requested?
├── Nothing missing?
├── Nothing extra?
└── GATE: Must pass before Stage 2

STAGE 2: CODE QUALITY
├── Clean, tested, maintainable?
├── Architecture and patterns?
├── Test coverage?
└── Output: Critical/Important/Minor
```

### Issue Severity Actions

| Severity | Action |
|----------|--------|
| **Critical** | Fix immediately, blocks progress |
| **Important** | Fix before proceeding |
| **Minor** | Note for later |

### Never Skip

- NEVER skip reviews because "it's simple"
- NEVER ignore Critical issues
- NEVER proceed with unfixed Important issues
- NEVER skip re-review after fixes
