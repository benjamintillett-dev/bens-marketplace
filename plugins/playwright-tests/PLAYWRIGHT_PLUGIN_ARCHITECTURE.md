# Playwright Testing Plugin - Comprehensive Orchestrated Architecture

**Goal**: Build a production-grade Claude Code plugin that coordinates multiple specialized sub-agents in parallel to fix Playwright tests 10x faster than manual debugging.

**Inspired By**: Compounding Engineering plugin (15 agents + 6 commands)

---

## Executive Summary

This plugin provides:
- **1 Orchestrator Agent** - Coordinates parallel execution
- **5 Specialized Sub-Agents** - Work simultaneously on different failure types
- **3 Auto-Activating Skills** - Trigger on test failure context
- **4 Slash Commands** - Manual control and workflows
- **6 Supporting Scripts** - Trace analysis, pattern detection, selector healing
- **2 Hooks** - Pre-commit test validation, CI failure notifications

**Expected Performance**:
- **Current**: 2-3 hours to fix 20 failing tests manually
- **With Plugin**: 15-20 minutes (85-90% reduction)
- **Parallel Speedup**: 5 agents working simultaneously = 5x throughput
- **Learning Over Time**: Pattern library grows with each fix

---

## Architecture Overview

### Hierarchical Multi-Agent System

```
User Request: "Fix the failing Playwright tests"
     ↓
[Orchestrator Agent] ← Analyzes failures, plans work distribution
     ↓
Spawns 5 Parallel Sub-Agents:
     ├─→ [Selector Fixer] ─→ Fixes strict mode violations (6 tests)
     ├─→ [Timing Optimizer] ─→ Adds networkidle waits (4 tests)
     ├─→ [Auto-Save Handler] ─→ Fixes debounce issues (4 tests)
     ├─→ [Data Cleaner] ─→ Adds .first() calls (3 tests)
     └─→ [Assertion Fixer] ─→ Fixes URL matching (3 tests)
     ↓
[Orchestrator] ← Collects results, runs verification
     ↓
Report: 20/20 tests fixed in 18 minutes
```

**Key Innovation**: Instead of sequential fixing (1 → 2 → 3 → ... → 20), the orchestrator batches failures by type and assigns each batch to a specialized agent working in parallel.

---

## Plugin Directory Structure

```
.claude-plugin/
└── plugin.json                    # Plugin metadata

agents/
├── orchestrator.md                # Main coordinator agent
├── selector-fixer.md              # Strict mode & selector issues
├── timing-optimizer.md            # Hydration & race conditions
├── autosave-handler.md            # Auto-save debounce issues
├── data-cleaner.md                # Test data accumulation
└── assertion-fixer.md             # URL & content assertions

skills/
├── test-failure-detector/
│   └── SKILL.md                   # Auto-activates on test output
├── trace-analyzer/
│   └── SKILL.md                   # Auto-activates on .zip traces
└── flaky-test-hunter/
    └── SKILL.md                   # Auto-activates on intermittent failures

commands/
├── fix-tests.md                   # Run orchestrator manually
├── analyze-flaky.md               # Detect flaky tests (10x runs)
├── heal-selectors.md              # Update Page Objects with better selectors
└── clean-test-data.md             # Database cleanup

scripts/
├── analyze-trace.ts               # Parse Playwright trace.zip files
├── detect-patterns.ts             # Categorize failures by root cause
├── suggest-fixes.ts               # Map error patterns → code fixes
├── selector-alternatives.ts       # Find robust selectors from screenshots
├── flaky-detector.ts              # Run tests N times, identify inconsistencies
└── test-health-report.ts          # Generate dashboard of test metrics

hooks/
└── hooks.json                     # Pre-commit hook for test validation

templates/
├── selector-fix.md                # Template for selector fixes
├── timing-fix.md                  # Template for timing fixes
├── autosave-fix.md                # Template for auto-save fixes
└── data-cleanup-fix.md            # Template for data cleanup

docs/
├── README.md                      # Plugin usage guide
├── ARCHITECTURE.md                # This file
└── TROUBLESHOOTING.md             # Common issues & solutions

test-data/
└── known-patterns.json            # Pattern library (grows over time)
```

---

## Component Specifications

### 1. Orchestrator Agent

**File**: `agents/orchestrator.md`

```yaml
---
name: playwright-orchestrator
description: Orchestrates parallel test fixing by analyzing failures, categorizing by root cause, and spawning specialized sub-agents to work simultaneously. Invoke when you need to fix multiple Playwright test failures efficiently.
tools:
  - Read
  - Bash
  - Grep
  - Glob
  - Task
model: sonnet
permissionMode: auto
---

# Playwright Test Orchestrator

You are the **master coordinator** for Playwright test fixing. Your job is to:
1. Analyze all test failures holistically
2. Categorize by root cause (selector, timing, auto-save, data, assertion)
3. Spawn specialized sub-agents to work in parallel
4. Collect results and verify fixes
5. Report comprehensive summary

## Workflow

### Step 1: Run Tests and Analyze (2 minutes)

```bash
pnpm test
```

Parse output and categorize failures:
- **Selector Issues** (strict mode, element not found)
- **Timing Issues** (timeout, hydration, networkidle)
- **Auto-Save Issues** (2s debounce race conditions)
- **Data Issues** (test data accumulation)
- **Assertion Issues** (URL matching, content checks)

Count failures per category.

### Step 2: Spawn Parallel Sub-Agents (10-15 minutes)

Based on failure counts, spawn sub-agents **in parallel** using Task tool:

```typescript
// Example: 6 selector issues, 4 timing issues, 4 auto-save issues, 3 data issues, 3 assertion issues

Task({
  subagent_type: "playwright:selector-fixer",
  prompt: "Fix 6 strict mode violations in these tests: [list]",
  description: "Fix selector issues",
  model: "haiku"
});

Task({
  subagent_type: "playwright:timing-optimizer",
  prompt: "Fix 4 timeout/hydration issues in these tests: [list]",
  description: "Fix timing issues",
  model: "haiku"
});

Task({
  subagent_type: "playwright:autosave-handler",
  prompt: "Fix 4 auto-save debounce issues in these tests: [list]",
  description: "Fix auto-save issues",
  model: "haiku"
});

Task({
  subagent_type: "playwright:data-cleaner",
  prompt: "Fix 3 test data accumulation issues in these tests: [list]",
  description: "Fix data issues",
  model: "haiku"
});

Task({
  subagent_type: "playwright:assertion-fixer",
  prompt: "Fix 3 assertion failures in these tests: [list]",
  description: "Fix assertion issues",
  model: "haiku"
});
```

**CRITICAL**: Invoke ALL sub-agents in a SINGLE message with multiple Task tool calls. This enables true parallel execution.

### Step 3: Monitor Progress (Real-time)

As each sub-agent completes:
- Track progress: "3/5 agents complete"
- Report fixes: "Selector agent fixed 6/6 tests"
- Handle failures: If agent fails, retry or escalate

### Step 4: Verify All Fixes (2 minutes)

After all agents complete:
```bash
pnpm test
```

Verify success rate. If failures remain:
- Analyze new errors (may have uncovered hidden issues)
- Spawn additional agents if needed
- Or handle manually for edge cases

### Step 5: Report Summary

```
🎯 Orchestration Complete

Initial State: 20 failing tests
  • Selector issues: 6
  • Timing issues: 4
  • Auto-save issues: 4
  • Data issues: 3
  • Assertion issues: 3

Agents Deployed (parallel):
  ✓ Selector Fixer → 6/6 tests fixed
  ✓ Timing Optimizer → 4/4 tests fixed
  ✓ Auto-Save Handler → 4/4 tests fixed
  ✓ Data Cleaner → 3/3 tests fixed
  ✓ Assertion Fixer → 3/3 tests fixed

Final State: 20 passing tests (100% success)

Time: 18 minutes
Speedup: 85% faster than manual (vs. 2-3 hours)
Parallel Efficiency: 5x throughput
```

## Sub-Agent Selection Logic

Choose sub-agent based on error patterns:

| Error Pattern | Sub-Agent | Model |
|--------------|-----------|-------|
| "strict mode violation", "resolved to 2+ elements" | selector-fixer | haiku |
| "Timeout", "waiting for", "exceeded" | timing-optimizer | haiku |
| "toContain" + create-entry.spec.ts | autosave-handler | haiku |
| Multiple entries with same text | data-cleaner | haiku |
| "toHaveURL", "toMatch", URL assertions | assertion-fixer | haiku |

Use `haiku` for sub-agents (fast + cheap), reserve `sonnet` for orchestrator (complex planning).

## Parallel Execution Strategy

**Key Insight**: Claude Code supports parallel Task tool calls in a single message.

```typescript
// ✅ CORRECT: Parallel execution (single message, multiple tasks)
// All 5 agents start simultaneously
await Promise.all([
  Task({ subagent_type: "selector-fixer", ... }),
  Task({ subagent_type: "timing-optimizer", ... }),
  Task({ subagent_type: "autosave-handler", ... }),
  Task({ subagent_type: "data-cleaner", ... }),
  Task({ subagent_type: "assertion-fixer", ... })
]);

// ❌ WRONG: Sequential execution (waits for each to finish)
await Task({ subagent_type: "selector-fixer", ... });
await Task({ subagent_type: "timing-optimizer", ... });
// ... etc
```

## Critical Rules

1. **ALWAYS** spawn sub-agents in parallel (single message with multiple Task calls)
2. **NEVER** spawn more than 5 agents simultaneously (context limits)
3. **ALWAYS** use haiku model for sub-agents (cost optimization)
4. **ALWAYS** verify fixes after all agents complete
5. **NEVER** assume - always run tests to confirm

## Success Criteria

- Fix 90%+ of tests in first orchestration pass
- Complete in 15-25 minutes (vs 2-3 hours manual)
- No context pollution in main conversation
- Parallel efficiency: 4-5x speedup vs sequential
```

---

### 2. Selector Fixer Sub-Agent

**File**: `agents/selector-fixer.md`

```yaml
---
name: playwright:selector-fixer
description: Specialized agent for fixing Playwright selector issues, strict mode violations, and element-not-found errors. Expert in semantic locators, data-testid attributes, and exact matching.
tools:
  - Read
  - Edit
  - Grep
  - Glob
model: haiku
permissionMode: auto
---

# Selector Fixer Agent

You are a **Playwright selector specialist**. Your only job is to fix selector-related test failures.

## Common Patterns

### Pattern 1: Strict Mode Violations
**Error**: "strict mode violation: getByRole('button', { name: /edit/i }) resolved to 2 elements"

**Root Cause**: Selector matches multiple elements (entry cards + action buttons)

**Fixes** (in order of preference):
1. **Add `.first()`** - Quick fix for lists
   ```typescript
   await expect(page.getByText('Entry').first()).toBeVisible();
   ```

2. **Use exact matching** - Prevent partial matches
   ```typescript
   page.getByRole('button', { name: 'Edit', exact: true })
   ```

3. **Use type attribute** - Distinguish buttons
   ```typescript
   page.locator('button[type="submit"]')
   ```

4. **Add data-testid** - Last resort for brittle selectors
   ```typescript
   // In component:
   <button data-testid="delete-entry-button">Delete</button>

   // In test:
   page.getByTestId('delete-entry-button')
   ```

### Pattern 2: Element Not Found
**Error**: "locator.click: Error: element not found"

**Root Cause**: Selector doesn't match any elements

**Fixes**:
1. **Check spelling** - Verify text matches exactly
2. **Check case sensitivity** - Use `/i` flag if needed
3. **Wait for element** - Add explicit wait
   ```typescript
   await page.getByRole('button', { name: 'Submit' }).waitFor({ state: 'visible' });
   ```

### Pattern 3: Dynamic Content
**Error**: Selector works locally but fails in CI

**Root Cause**: Content loads asynchronously

**Fix**: Use waitFor with timeout
```typescript
await expect(page.getByText('Loaded content')).toBeVisible({ timeout: 10000 });
```

## Workflow

1. **Read test file** - Understand what selector is failing
2. **Read Page Object** (if using POM pattern) - See selector definition
3. **Apply fix** using Edit tool - Surgical change only
4. **Output fix details** - Report what changed

## Output Format

```
🔧 Selector Fix: [Test Name]
   File: [path:line]
   Error: [brief error]
   Root Cause: [diagnosis]
   Fix Applied: [description]

   Changed:
   - [file:line] Added .first() to entry assertion
```

## Critical Rules

1. **ONLY** fix selector issues - don't touch timing, assertions, etc.
2. **ALWAYS** use exact string matching in Edit tool
3. **NEVER** add data-testid unless absolutely necessary (keep tests semantic)
4. **ALWAYS** prefer semantic locators (getByRole, getByLabel) over CSS selectors
```

---

### 3. Timing Optimizer Sub-Agent

**File**: `agents/timing-optimizer.md`

```yaml
---
name: playwright:timing-optimizer
description: Specialized agent for fixing Playwright timing issues, timeouts, hydration race conditions, and networkidle waits. Expert in Svelte 5 SSR hydration patterns.
tools:
  - Read
  - Edit
  - Grep
  - Glob
model: haiku
permissionMode: auto
---

# Timing Optimizer Agent

You are a **Playwright timing specialist**. Your only job is to fix timing-related test failures.

## Common Patterns

### Pattern 1: Svelte 5 Hydration Race Condition
**Error**: "Timeout 5000ms exceeded waiting for button click" (first interaction only)

**Root Cause**: Playwright clicks before JavaScript hydrates event handlers

**Fix**: Add `waitUntil: 'networkidle'` to page navigation
```typescript
// In Page Object Model goto() methods:
async goto() {
  await this.page.goto('/login', { waitUntil: 'networkidle' });
}
```

**Symptoms**:
- Test fails on first interaction after page load
- Works when run individually but fails in suite
- Click/fill actions have no effect

### Pattern 2: Generic Timeouts
**Error**: "Timeout 5000ms exceeded"

**Root Cause**: Default timeout too short for slow operations

**Fixes**:
1. **Increase timeout** for specific action
   ```typescript
   await page.getByRole('button').click({ timeout: 10000 });
   ```

2. **Wait for condition** before action
   ```typescript
   await page.getByRole('button').waitFor({ state: 'visible', timeout: 10000 });
   await page.getByRole('button').click();
   ```

### Pattern 3: Navigation Timing
**Error**: "waitForURL timed out"

**Root Cause**: Form submission or navigation slower than expected

**Fix**: Increase navigation timeout
```typescript
await page.waitForURL('/pages', { timeout: 15000 });
```

### Pattern 4: Button Toggle State Timing
**Error**: Button text doesn't change after click

**Root Cause**: Svelte 5 reactive updates not completed

**Fix**: Wait for specific state change
```typescript
await page.getByRole('button', { name: /toggle/i }).click();
await expect(page.locator('form')).toHaveAttribute('action', '?/signup', { timeout: 5000 });
```

## Workflow

1. **Read test file** - Identify timing failure
2. **Analyze context** - Determine if hydration, network, or reactive update
3. **Apply appropriate fix** - Use pattern above
4. **Report fix** - Output details

## Output Format

```
⏱️ Timing Fix: [Test Name]
   File: [path:line]
   Error: [timeout error]
   Root Cause: [Svelte 5 hydration / slow navigation / etc]
   Fix Applied: [Added networkidle wait / increased timeout / etc]

   Changed:
   - [file:line] Added waitUntil: 'networkidle' to goto()
```

## Critical Rules

1. **ONLY** fix timing issues - don't touch selectors, assertions, etc.
2. **ALWAYS** consider Svelte 5 hydration first for post-navigation failures
3. **PREFER** networkidle over arbitrary timeouts when possible
4. **NEVER** add long hardcoded delays (e.g., waitForTimeout(5000)) unless necessary
```

---

### 4. Auto-Save Handler Sub-Agent

**File**: `agents/autosave-handler.md`

```yaml
---
name: playwright:autosave-handler
description: Specialized agent for fixing auto-save timing issues in WriteEditor component. Expert in 2-second debounce patterns and save verification strategies.
tools:
  - Read
  - Edit
  - Grep
model: haiku
permissionMode: auto
---

# Auto-Save Handler Agent

You are an **auto-save timing specialist** for the WriteEditor component.

## Context

The WriteEditor component has:
- **2-second debounce** on content changes
- **Auto-save** to Supabase after debounce
- **No visual indicator** when save completes

## Common Pattern: Auto-Save Race Condition

**Error**: "Expected content to contain [text] but got [empty/different]"

**Location**: create-entry.spec.ts, edit-entry.spec.ts

**Root Cause**: Test checks content before 2s debounce + network request completes

**Fix**: Add explicit wait after content input
```typescript
await page.getByTestId('editor-textarea').fill(content);
await page.waitForTimeout(3000); // 2s debounce + 1s network buffer
```

**Alternative Fix** (if save indicator exists):
```typescript
await page.getByTestId('editor-textarea').fill(content);
await expect(page.getByText(/saved/i)).toBeVisible({ timeout: 5000 });
```

## Workflow

1. **Identify auto-save test** - Tests in create-entry.spec.ts or edit-entry.spec.ts
2. **Check for wait** - Does test wait for auto-save?
3. **Add 3s wait** - After fill() action
4. **Report fix**

## Output Format

```
💾 Auto-Save Fix: [Test Name]
   File: [path:line]
   Issue: Content assertion failed due to auto-save timing
   Fix Applied: Added 3s wait for 2s debounce + network

   Changed:
   - [file:line] Added await page.waitForTimeout(3000) after fill()
```

## Critical Rules

1. **ONLY** fix auto-save timing issues
2. **ALWAYS** use 3000ms minimum (2s debounce + 1s buffer)
3. **NEVER** reduce timeout below 3s (will cause flakiness)
```

---

### 5. Data Cleaner Sub-Agent

**File**: `agents/data-cleaner.md`

```yaml
---
name: playwright:data-cleaner
description: Specialized agent for fixing test data accumulation issues. Expert in adding .first() calls and identifying data pollution patterns.
tools:
  - Read
  - Edit
  - Grep
model: haiku
permissionMode: auto
---

# Data Cleaner Agent

You are a **test data specialist**. Your job is to fix issues caused by test data accumulation.

## Common Pattern: Duplicate Entries

**Error**: "strict mode violation: getByText('Test Entry') resolved to 3 elements"

**Root Cause**: Tests create entries but don't clean up, so multiple runs create duplicates

**Immediate Fix**: Add `.first()` to selectors
```typescript
await expect(page.getByText('Test Entry').first()).toBeVisible();
```

**Long-term Recommendation**: Implement cleanup fixtures (note for later)

## Workflow

1. **Identify data accumulation** - Error mentions "resolved to N elements"
2. **Add `.first()`** - To all affected selectors in test
3. **Report fix** - Include recommendation for fixtures

## Output Format

```
🗑️ Data Cleanup Fix: [Test Name]
   File: [path:line]
   Issue: Multiple entries from previous test runs
   Fix Applied: Added .first() to handle duplicates

   ⚠️ Recommendation: Implement cleanup fixtures to prevent accumulation

   Changed:
   - [file:line] Added .first() to entry visibility check
```

## Critical Rules

1. **ONLY** fix data accumulation issues
2. **ALWAYS** add .first() to handle duplicates
3. **ALWAYS** recommend fixtures as long-term solution
```

---

### 6. Assertion Fixer Sub-Agent

**File**: `agents/assertion-fixer.md`

```yaml
---
name: playwright:assertion-fixer
description: Specialized agent for fixing assertion failures, URL matching issues, and content verification problems. Expert in SvelteKit form action patterns.
tools:
  - Read
  - Edit
model: haiku
permissionMode: auto
---

# Assertion Fixer Agent

You are an **assertion specialist**. Your job is to fix assertion-related test failures.

## Common Patterns

### Pattern 1: SvelteKit Form Action URLs
**Error**: "Expected URL to be '/login' but was '/login?/signup'"

**Root Cause**: SvelteKit adds query params for form actions

**Fix**: Use regex instead of exact match
```typescript
// ❌ Wrong
await expect(page).toHaveURL('/login');

// ✅ Correct
await expect(page.url()).toMatch(/\/login/);
```

### Pattern 2: Content Assertions
**Error**: "Expected text to contain [expected] but got [actual]"

**Root Cause**: Content format different than expected (whitespace, case, etc.)

**Fixes**:
1. **Use regex** for flexible matching
   ```typescript
   await expect(element).toHaveText(/expected text/i);
   ```

2. **Use toContain** instead of exact match
   ```typescript
   await expect(element).toContainText('expected');
   ```

## Workflow

1. **Identify assertion failure** - What assertion failed?
2. **Analyze expected vs actual** - Why did it fail?
3. **Apply appropriate fix** - Regex, toContain, or adjust expectation
4. **Report fix**

## Output Format

```
✓ Assertion Fix: [Test Name]
   File: [path:line]
   Failed Assertion: [assertion type]
   Root Cause: [why it failed]
   Fix Applied: [description]

   Changed:
   - [file:line] Changed toHaveURL to .toMatch(/\/login/)
```

## Critical Rules

1. **ONLY** fix assertion issues
2. **PREFER** flexible matchers (regex, toContain) over exact matches
3. **NEVER** change assertions to make tests pass incorrectly (fix root cause)```

---

## Skills (Auto-Activating Context Providers)

### 7. Test Failure Detector Skill

**File**: `skills/test-failure-detector/SKILL.md`

```markdown
---
name: test-failure-detector
description: Automatically activated when Playwright test output contains failures, errors, or timeouts. Provides instant pattern analysis and suggests next actions.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
---

# Test Failure Detector Skill

I automatically activate when I detect Playwright test failures in the conversation.

## Activation Triggers

- Test output containing "X passing", "Y failing"
- Error stack traces from Playwright
- Timeout messages
- Selector failures
- Assertion failures

## What I Do

When activated, I:
1. Parse test output to extract failure details
2. Categorize failures by pattern (selector, timing, assertion, etc.)
3. Count failures per category
4. Suggest invoking the orchestrator agent

## Output

```
🔍 Test Failure Detected

Total Failures: 20
Categorized:
  • Selector issues: 6 tests
  • Timing issues: 4 tests
  • Auto-save issues: 4 tests
  • Data issues: 3 tests
  • Assertion issues: 3 tests

💡 Recommendation: Invoke the Playwright Orchestrator to fix these in parallel.

Would you like me to start the orchestrator?
```
```

---

### 8. Trace Analyzer Skill

**File**: `skills/trace-analyzer/SKILL.md`

```markdown
---
name: trace-analyzer
description: Automatically activated when Playwright trace files (.zip) are mentioned or when trace analysis is needed. Provides insights into test execution timeline and bottlenecks.
allowed-tools:
  - Read
  - Bash
---

# Trace Analyzer Skill

I automatically activate when Playwright traces are mentioned.

## Activation Triggers

- Mention of "trace.zip" files
- Discussion of test slowness
- Request to analyze test execution
- CI failure with trace artifacts

## What I Do

1. Parse trace.zip files using scripts/analyze-trace.ts
2. Extract timeline: navigations, network calls, assertions
3. Identify bottlenecks (e.g., "70% of time spent waiting for auto-save")
4. Suggest optimizations

## Output

```
📊 Trace Analysis: create-entry.spec.ts

Timeline:
  00:00 - 00:50  Navigate to /write (50ms)
  00:50 - 01:00  Fill textarea (50ms)
  01:00 - 04:00  ⚠️ Wait for auto-save (3000ms) ← BOTTLENECK (70% of test time)
  04:00 - 04:20  Navigate to /pages (200ms)
  04:20 - 04:30  Assert entry visible (100ms)

Total: 4.3 seconds

💡 Optimization Opportunities:
  1. Mock Supabase for deterministic saves (eliminates 3s wait)
  2. Add save indicator to avoid arbitrary waits
  3. Run create-entry tests with mocked backend
```
```

---

### 9. Flaky Test Hunter Skill

**File**: `skills/flaky-test-hunter/SKILL.md`

```markdown
---
name: flaky-test-hunter
description: Automatically activated when tests pass/fail intermittently or when flakiness is mentioned. Runs tests multiple times to detect inconsistencies and categorizes root causes.
allowed-tools:
  - Bash
  - Read
---

# Flaky Test Hunter Skill

I automatically activate when test flakiness is mentioned.

## Activation Triggers

- Mention of "flaky tests"
- Tests passing locally but failing in CI
- Intermittent failures
- Request to detect flakiness

## What I Do

1. Run suspect tests 10-20 times using scripts/flaky-detector.ts
2. Track pass/fail rate for each test
3. Categorize by root cause (timing, network, race condition)
4. Suggest targeted fixes

## Output

```
🔎 Flaky Test Detection

Ran each test 10 times:

Stable Tests (100% pass rate):
  ✓ auth.spec.ts: should login successfully (10/10)
  ✓ view-entries.spec.ts: should display entries (10/10)

Flaky Tests:
  ⚠️ create-entry.spec.ts: should auto-save (6/10 pass) - 40% failure rate
     Root Cause: Auto-save timing (2s debounce + network latency)
     Fix: Add explicit 3s wait or mock Supabase

  ⚠️ delete-entry.spec.ts: should delete entry (8/10 pass) - 20% failure rate
     Root Cause: Navigation timing after delete action
     Fix: Wait for networkidle or add URL assertion with timeout

💡 Recommendation: Fix these 2 flaky tests first (highest impact)
```
```

---

## Commands (Manual Workflows)

### 10. /fix-tests Command

**File**: `commands/fix-tests.md`

```markdown
---
description: Manually invoke the Playwright Orchestrator to fix all failing tests in parallel. Analyzes failures, spawns specialized sub-agents, and verifies fixes.
---

# Fix Tests Command

Manually invoke the Playwright Orchestrator agent to fix failing tests.

## What It Does

1. Runs `pnpm test` to get current failures
2. Analyzes and categorizes failures
3. Spawns 5 specialized sub-agents in parallel
4. Verifies all fixes
5. Reports comprehensive summary

## Usage

```
/fix-tests
```

## When to Use

- After making code changes that broke tests
- When you have multiple failing tests
- When you want parallel fixing (faster than manual)

## Expected Time

- 15-25 minutes for 15-25 failing tests
- Scales linearly with test count
```

---

### 11. /analyze-flaky Command

**File**: `commands/analyze-flaky.md`

```markdown
---
description: Run all tests multiple times (default: 10x) to detect flaky tests and categorize by root cause. Provides targeted fix suggestions.
---

# Analyze Flaky Tests Command

Detect flaky tests by running them multiple times.

## What It Does

1. Runs entire test suite N times (default: 10)
2. Tracks pass/fail rate for each test
3. Identifies tests with < 100% pass rate
4. Categorizes by root cause
5. Suggests targeted fixes

## Usage

```
/analyze-flaky
/analyze-flaky --runs 20
/analyze-flaky --test "create-entry.spec.ts"
```

## When to Use

- Tests pass locally but fail in CI
- Intermittent failures
- After refactoring timing-sensitive code
- Before deploying to production

## Expected Time

- 20-40 minutes for 10 runs of full suite
- Use --test flag to focus on specific files
```

---

### 12. /heal-selectors Command

**File**: `commands/heal-selectors.md`

```markdown
---
description: Analyze Page Objects and suggest more robust selectors. Uses screenshots and DOM analysis to find semantic alternatives to brittle selectors.
---

# Heal Selectors Command

Update Page Objects with more robust selectors.

## What It Does

1. Analyzes all Page Object files
2. Identifies brittle selectors (CSS classes, data-testid)
3. Suggests semantic alternatives (getByRole, getByLabel)
4. Uses scripts/selector-alternatives.ts
5. Applies changes with user approval

## Usage

```
/heal-selectors
/heal-selectors --page LoginPage
```

## When to Use

- Tests break after UI refactoring
- Too many data-testid attributes
- Want to improve test maintainability

## Example Output

```
🔧 Selector Healing: LoginPage.ts

Current (Brittle):
  page.locator('[data-testid="email-input"]')

Suggested (Semantic):
  page.getByLabel(/email/i)

Reason: Label exists and provides better semantics

Apply change? (y/n)
```
```

---

### 13. /clean-test-data Command

**File**: `commands/clean-test-data.md`

```markdown
---
description: Clean up accumulated test data in the database. Deletes all entries created by test user to prevent test data accumulation.
---

# Clean Test Data Command

Remove accumulated test data from database.

## What It Does

1. Connects to Supabase using test credentials
2. Deletes all journal entries for test user
3. Reports how many entries removed
4. Optionally resets test user state

## Usage

```
/clean-test-data
/clean-test-data --user test@example.com
```

## When to Use

- Before running tests locally
- When tests fail due to data accumulation
- After test suite refactoring

## Safety

- Only deletes data for test users (email contains "test")
- Never touches production data
- Requires confirmation before deletion
```

---

## Supporting Scripts

### 14. Trace Analyzer Script

**File**: `scripts/analyze-trace.ts`

```typescript
import { parseTraceZip } from '@playwright/test/trace-parser';
import { readFileSync } from 'fs';

interface TraceEvent {
  type: 'navigation' | 'action' | 'assertion' | 'wait';
  timestamp: number;
  duration: number;
  description: string;
}

export async function analyzeTrace(zipPath: string): Promise<{
  events: TraceEvent[];
  bottlenecks: { event: TraceEvent; percentage: number }[];
  totalDuration: number;
}> {
  // Parse trace.zip file
  const trace = await parseTraceZip(zipPath);
  
  // Extract timeline events
  const events: TraceEvent[] = [];
  let totalDuration = 0;

  for (const action of trace.actions) {
    const event: TraceEvent = {
      type: action.type as any,
      timestamp: action.startTime,
      duration: action.endTime - action.startTime,
      description: action.apiName
    };
    events.push(event);
    totalDuration += event.duration;
  }

  // Identify bottlenecks (>20% of total time)
  const bottlenecks = events
    .map(event => ({
      event,
      percentage: (event.duration / totalDuration) * 100
    }))
    .filter(b => b.percentage > 20)
    .sort((a, b) => b.percentage - a.percentage);

  return { events, bottlenecks, totalDuration };
}
```

### 15. Pattern Detector Script

**File**: `scripts/detect-patterns.ts`

```typescript
interface TestFailure {
  testName: string;
  errorMessage: string;
  location: string;
}

interface FailurePattern {
  category: 'selector' | 'timing' | 'autosave' | 'data' | 'assertion';
  count: number;
  tests: string[];
  suggestedFix: string;
}

export function detectPatterns(failures: TestFailure[]): FailurePattern[] {
  const patterns: Map<string, TestFailure[]> = new Map();

  for (const failure of failures) {
    const category = categorizeFailure(failure.errorMessage);
    const key = category;
    
    if (!patterns.has(key)) {
      patterns.set(key, []);
    }
    patterns.get(key)!.push(failure);
  }

  return Array.from(patterns.entries()).map(([category, tests]) => ({
    category: category as any,
    count: tests.length,
    tests: tests.map(t => t.testName),
    suggestedFix: getSuggestedFix(category as any, tests[0].errorMessage)
  }));
}

function categorizeFailure(error: string): string {
  if (error.includes('strict mode violation')) return 'selector';
  if (error.includes('Timeout')) return 'timing';
  if (error.includes('create-entry') && error.includes('toContain')) return 'autosave';
  if (error.includes('resolved to')) return 'data';
  return 'assertion';
}

function getSuggestedFix(category: string, error: string): string {
  const fixes = {
    selector: 'Add .first() or use exact matching',
    timing: 'Add networkidle wait or increase timeout',
    autosave: 'Add 3s wait for 2s debounce + network',
    data: 'Add .first() and implement cleanup fixtures',
    assertion: 'Use regex matching or toContain'
  };
  return fixes[category] || 'Manual investigation required';
}
```

### 16. Flaky Test Detector Script

**File**: `scripts/flaky-detector.ts`

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

interface FlakyTestResult {
  testName: string;
  passCount: number;
  failCount: number;
  passRate: number;
  isFlaky: boolean;
  rootCause?: string;
}

export async function detectFlakyTests(
  runs: number = 10,
  testPattern?: string
): Promise<FlakyTestResult[]> {
  const results = new Map<string, { passes: number; fails: number }>();

  for (let i = 0; i < runs; i++) {
    console.log(`Run ${i + 1}/${runs}...`);
    
    const command = testPattern 
      ? `pnpm test ${testPattern}`
      : 'pnpm test';

    try {
      const { stdout } = await execAsync(command);
      parseTestOutput(stdout, results, true);
    } catch (error: any) {
      parseTestOutput(error.stdout, results, false);
    }
  }

  return Array.from(results.entries()).map(([testName, counts]) => {
    const passRate = counts.passes / runs;
    return {
      testName,
      passCount: counts.passes,
      failCount: counts.fails,
      passRate,
      isFlaky: passRate < 1.0 && passRate > 0,
      rootCause: passRate < 1.0 ? guessRootCause(testName) : undefined
    };
  }).filter(r => r.isFlaky);
}

function parseTestOutput(
  output: string,
  results: Map<string, { passes: number; fails: number }>,
  passed: boolean
): void {
  const testNameRegex = /›\s+(.+)/g;
  let match;

  while ((match = testNameRegex.exec(output)) !== null) {
    const testName = match[1];
    if (!results.has(testName)) {
      results.set(testName, { passes: 0, fails: 0 });
    }
    if (passed) {
      results.get(testName)!.passes++;
    } else {
      results.get(testName)!.fails++;
    }
  }
}

function guessRootCause(testName: string): string {
  if (testName.includes('auto-save') || testName.includes('create')) {
    return 'Auto-save timing (2s debounce + network latency)';
  }
  if (testName.includes('delete') || testName.includes('navigate')) {
    return 'Navigation timing after action';
  }
  if (testName.includes('login') || testName.includes('auth')) {
    return 'Hydration race condition';
  }
  return 'Unknown - manual investigation required';
}
```

---

## Implementation Plan

### Phase 1: Core Infrastructure (Week 1)

**Goal**: Get orchestrator + 5 sub-agents working

#### Day 1-2: Plugin Setup
- [ ] Create `.claude-plugin/plugin.json`
- [ ] Set up directory structure
- [ ] Write orchestrator agent (most critical)
- [ ] Test orchestrator with 1 sub-agent

#### Day 3-4: Sub-Agents
- [ ] Write 5 specialized sub-agents
- [ ] Test each sub-agent individually
- [ ] Verify parallel execution works

#### Day 5-7: Integration & Testing
- [ ] Test orchestrator + all sub-agents on current 20 failures
- [ ] Measure time savings
- [ ] Refine agent prompts based on results
- [ ] Document usage patterns

**Success Criteria**:
- ✅ Orchestrator spawns 5 agents in parallel
- ✅ Fix 15-20 tests in 15-25 minutes
- ✅ 80%+ success rate on first pass

---

### Phase 2: Skills & Auto-Activation (Week 2)

**Goal**: Add auto-activating skills for proactive debugging

#### Day 8-9: Test Failure Detector Skill
- [ ] Write skill definition
- [ ] Test auto-activation on test output
- [ ] Integrate with orchestrator (skill suggests running it)

#### Day 10-11: Trace Analyzer Skill
- [ ] Write skill definition
- [ ] Implement analyze-trace.ts script
- [ ] Test on real trace files

#### Day 12-14: Flaky Test Hunter Skill
- [ ] Write skill definition
- [ ] Implement flaky-detector.ts script
- [ ] Test on known flaky tests

**Success Criteria**:
- ✅ Skills activate automatically based on context
- ✅ Trace analysis identifies bottlenecks
- ✅ Flaky test detection works reliably

---

### Phase 3: Commands & Scripts (Week 3)

**Goal**: Add manual control and supporting utilities

#### Day 15-16: Commands
- [ ] /fix-tests command (wrapper for orchestrator)
- [ ] /analyze-flaky command
- [ ] /heal-selectors command
- [ ] /clean-test-data command

#### Day 17-19: Scripts
- [ ] detect-patterns.ts
- [ ] selector-alternatives.ts
- [ ] suggest-fixes.ts
- [ ] test-health-report.ts

#### Day 20-21: Integration
- [ ] Test all commands end-to-end
- [ ] Verify scripts work with commands
- [ ] Write comprehensive README

**Success Criteria**:
- ✅ All commands work end-to-end
- ✅ Scripts integrate smoothly
- ✅ Documentation complete

---

### Phase 4: Hooks & CI Integration (Week 4)

**Goal**: Automate with hooks and CI integration

#### Day 22-23: Pre-Commit Hook
- [ ] hooks/hooks.json configuration
- [ ] Run tests before commit
- [ ] Block commit if tests fail
- [ ] Suggest running orchestrator

#### Day 24-25: CI Failure Notifications
- [ ] Hook into GitHub Actions
- [ ] Auto-comment on PR with failure analysis
- [ ] Suggest fixes based on patterns

#### Day 26-28: Optimization & Polish
- [ ] Performance tuning
- [ ] Error handling improvements
- [ ] Add pattern library (known-patterns.json)
- [ ] Learning over time implementation

**Success Criteria**:
- ✅ Pre-commit hook prevents broken commits
- ✅ CI auto-comments with failure analysis
- ✅ Pattern library grows with each fix

---

## Expected Performance Metrics

### Time Savings

| Scenario | Manual Time | Plugin Time | Savings |
|----------|-------------|-------------|---------|
| Fix 20 failing tests | 2-3 hours | 15-25 min | 85-90% |
| Detect flaky tests | 1 hour | 5 min | 92% |
| Analyze trace files | 30 min | 2 min | 93% |
| Update selectors | 45 min | 5 min | 89% |
| Clean test data | 10 min | 30 sec | 95% |

**Total weekly savings**: 10-15 hours → 2-3 hours (80% reduction)

### Parallel Efficiency

- **Sequential fixing**: 20 tests × 6 min/test = 120 minutes
- **Parallel fixing**: 20 tests ÷ 5 agents × 6 min = 24 minutes
- **Speedup**: 5x throughput

### Cost Optimization

- **Orchestrator**: Sonnet (complex planning)
- **Sub-agents**: Haiku (simple fixes)
- **Cost per fix**: ~$0.05 (vs. developer time: $50-100/hour)
- **ROI**: 1000-2000x

---

## Conclusion

This comprehensive plugin architecture provides:

1. **Orchestrated Parallel Execution**: 5 specialized sub-agents working simultaneously
2. **Auto-Activating Skills**: Proactive debugging based on context
3. **Manual Control Commands**: Slash commands for explicit workflows
4. **Supporting Scripts**: Trace analysis, pattern detection, flaky test detection
5. **CI Integration**: Hooks for automated test validation

**Key Innovations**:
- **Parallel speedup**: 5x throughput vs sequential
- **Context isolation**: No pollution of main conversation
- **Learning over time**: Pattern library grows with each fix
- **Cost optimized**: Haiku for sub-agents, Sonnet for orchestrator

**Expected Outcome**:
- **85-90% time savings** on test debugging
- **15-25 minutes** to fix 20 tests (vs. 2-3 hours)
- **Zero context pollution** (all work in sub-agents)
- **Reusable across projects** (with minor config changes)

**Next Steps**:
1. Review this architecture with team
2. Start Phase 1 implementation (Week 1)
3. Test with current 20 failing tests
4. Iterate based on real-world usage

---

**Last Updated**: 2025-01-18
**Version**: 1.0
**Author**: AI-Assisted Architecture Design
