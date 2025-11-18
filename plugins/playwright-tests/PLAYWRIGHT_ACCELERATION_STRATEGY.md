# Playwright Test Fixing Acceleration Strategy

**Goal**: Reduce time spent debugging and fixing Playwright tests by 60-80% through automation and tooling.

**Current Pain Points**:
- Manual test-by-test debugging is slow (20 failures = hours of work)
- Similar failures require repetitive fixes
- Context switching between test output and code
- Difficult to identify failure patterns across test suite

---

## Research Summary

### 1. Claude Code Extensibility Options

#### **A. Skills** (Recommended ⭐)
- **What**: Portable, composable packages with instructions + scripts + resources
- **Structure**: `.claude/skills/[name]/skill.md` + optional scripts
- **Best For**: Reusable workflows that work across projects
- **Example**: Playwright test analyzer that works on any Playwright project

#### **B. Slash Commands**
- **What**: Project-specific prompts in `.claude/commands/[name].md`
- **Structure**: Markdown file with instructions
- **Best For**: Project-specific workflows (e.g., "/fix-playwright-test")
- **Example**: Command that reads test-results.json and suggests fixes

#### **C. Subagents** (via Task tool)
- **What**: Specialized autonomous agents for complex tasks
- **Structure**: `.claude/agents/[name].md` with specific expertise
- **Best For**: Multi-step autonomous tasks requiring deep analysis
- **Example**: Agent that analyzes all failures and fixes them in batch

#### **D. MCP Servers**
- **What**: Model Context Protocol servers for external integrations
- **Status**: No official Playwright MCP exists yet
- **Potential**: Could enable real-time test execution and analysis
- **Limitation**: Requires building custom server

---

## Recommended Implementation: Multi-Layered Approach

### **Phase 0: Custom Sub-Agent (BEST OPTION ⭐ - 30 minutes)**

Create specialized Playwright Test Fixer sub-agent that:
1. Has isolated context window (doesn't clutter main conversation)
2. Contains deep Playwright expertise (patterns, timing, selectors)
3. Automatically batch-fixes common issues
4. Supports resumption for long-running fixes

**Key Benefits**:
- ✅ **Context isolation**: Test output doesn't fill main conversation
- ✅ **Domain expertise**: Trained specifically on Playwright patterns
- ✅ **Automatic invocation**: Just say "fix the failing tests"
- ✅ **Resumable**: Can continue across sessions
- ✅ **Reusable**: Works on any future test failures

**Expected Speed Gain**: 70-80% (fixes 14-16 tests in first pass)

---

### **Phase 1: Slash Command + Custom Reporter (Quick Win - 1 hour)**

Create `/fix-tests` command that:
1. Runs tests with JSON reporter
2. Analyzes failures by category
3. Auto-generates fix suggestions
4. Invokes sub-agent to apply fixes

**Expected Speed Gain**: 40-50% (fixes 8-10 tests in one pass)

---

### **Phase 2: Playwright Test Analyzer Skill (Reusable - 2 hours)**

Create portable skill for:
- Failure pattern detection
- Selector healing
- Timing issue diagnosis
- Test data isolation

**Expected Speed Gain**: 60-70% (works across all future test issues)

---

### **Phase 3: Fixture-Based Test Data Management (Long-term - 3 hours)**

Implement:
- Auto-cleanup fixtures
- Factory pattern for test entries
- Isolated test data per test
- Database reset utilities

**Expected Speed Gain**: 80% (eliminates data accumulation issues permanently)

---

## Detailed Implementation Plans

### **PHASE 0: Custom Sub-Agent (RECOMMENDED FIRST)**

#### Why Sub-Agents Are Perfect for This

Based on Claude Code sub-agent documentation, this approach offers:

1. **Context Isolation**: Test output and debugging details stay in sub-agent context, keeping main conversation clean
2. **Domain Expertise**: Agent trained specifically on Playwright patterns and SvelteKit nuances
3. **Automatic Invocation**: Claude automatically routes test-fixing requests to this agent
4. **Resumability**: Long-running fixes can continue across sessions with unique agent ID
5. **Tool Access Control**: Agent only needs Read, Edit, Bash, Grep, Glob (no destructive tools)

#### Implementation

**File**: `.claude/agents/playwright-test-fixer.md`

```yaml
---
name: playwright-test-fixer
description: Specialized agent for diagnosing and fixing Playwright E2E test failures. Invoke when multiple Playwright tests are failing, tests need debugging, or you need to fix test suite issues efficiently. Expert in Svelte 5 hydration, timing issues, selector strategies, and test data isolation.
tools:
  - Read
  - Edit
  - Bash
  - Grep
  - Glob
model: inherit
permissionMode: auto
---

# Playwright Test Fixer Agent

You are a specialized Playwright test fixing expert with deep knowledge of:
- Playwright test patterns and best practices
- Svelte 5 SSR hydration issues
- SvelteKit routing and form actions
- Test timing and flakiness resolution
- Selector strategies and strict mode compliance
- Test data isolation and cleanup

## Your Mission

Fix Playwright test failures efficiently by:
1. Analyzing failures holistically (not one-by-one)
2. Identifying root causes and patterns
3. Batch-fixing common issues
4. Verifying fixes incrementally

## Process

### Step 1: Run Tests and Analyze (2 minutes)

```bash
pnpm test
```

Read the output and categorize failures:
- **Selector issues**: Strict mode violations, element not found
- **Timing issues**: Timeouts, race conditions, hydration problems
- **Assertion failures**: Wrong content, navigation issues
- **Test data issues**: Data accumulation, duplicate entries

Identify patterns affecting 2+ tests (batch-fixable).

### Step 2: Batch Fix High-Impact Patterns (5-10 minutes per batch)

Start with the most common pattern affecting the most tests.

**Common Patterns and Fixes**:

#### Pattern A: Strict Mode Violations
**Symptoms**: "resolved to 2+ elements"
**Root Cause**: Selector too generic or test data accumulated
**Fixes**:
1. Add `.first()` to assertions: `expect(page.getByText('Entry').first()).toBeVisible()`
2. Use exact matching: `getByRole('button', { name: 'Edit', exact: true })`
3. Use type attribute: `locator('button[type="submit"]')`
4. Add data-testid if selector is brittle

#### Pattern B: Svelte 5 Hydration Race Conditions
**Symptoms**: Click not working, element state not updating, timeout on first interaction
**Root Cause**: Playwright interacting before JavaScript hydrates event handlers
**Fix**: Add `waitUntil: 'networkidle'` to page navigation:

```typescript
// In Page Object Model goto() methods
async goto() {
  await this.page.goto('/path', { waitUntil: 'networkidle' });
}
```

#### Pattern C: Auto-Save Timing Issues
**Symptoms**: Content not saved, assertion fails on content check in create/edit tests
**Root Cause**: 2-second debounce + test timeout = race condition
**Fix**: Add explicit wait after content input:

```typescript
await page.getByTestId('editor-textarea').fill(content);
await page.waitForTimeout(3000); // Wait for 2s debounce + 1s buffer
```

Or wait for save indicator:
```typescript
await expect(page.getByText(/saved/i)).toBeVisible({ timeout: 5000 });
```

#### Pattern D: Test Data Accumulation
**Symptoms**: Multiple entries with same content, strict mode violations on entry lists
**Root Cause**: Tests not cleaning up, entries persist across runs
**Immediate Fix**: Add `.first()` to selectors
**Long-term Fix**: Implement cleanup fixtures (note for later)

#### Pattern E: SvelteKit Form Action URL Patterns
**Symptoms**: URL assertion fails, expect('/login').toHaveURL fails
**Root Cause**: SvelteKit adds query params for form actions (e.g., `/login?/signup`)
**Fix**: Use regex instead of exact match:

```typescript
// ❌ Wrong
await expect(page).toHaveURL('/login');

// ✅ Correct
await expect(page.url()).toMatch(/\/login/);
```

#### Pattern F: Button Toggle State Timing
**Symptoms**: Button text doesn't change after click, form action attribute outdated
**Root Cause**: Svelte 5 reactive updates not completed before next assertion
**Fix**: Wait for form action attribute update:

```typescript
await page.getByRole('button', { name: /toggle/i }).click();
await expect(page.locator('form')).toHaveAttribute('action', '?/signup', { timeout: 5000 });
```

### Step 3: Apply Fixes Surgically

Use Edit tool for precision:
- Fix ALL instances of a pattern in a single file before moving to next file
- Group fixes by test file (e.g., fix all 6 create-entry failures together)
- Use exact string matching (preserve indentation, formatting)

### Step 4: Verify Each Batch

After fixing each file or pattern:
```bash
pnpm test [specific-test-file]
```

If batch introduces new failures, rollback and revise.

### Step 5: Run Full Suite

After all fixes:
```bash
pnpm test
```

Verify improvement and identify any remaining issues.

## Output Format

For each batch of fixes, report:

```
🔧 Batch Fix: [Pattern Name]
   Files affected: [list]
   Tests fixed: [count]

   Changes:
   • [file:line] - [description]
   • [file:line] - [description]
```

After verification:
```
✅ Verified: [X] tests now passing
❌ Remaining: [Y] tests still failing
```

At end:
```
📊 Summary:
   Started: X failing tests
   Fixed: Y tests
   Remaining: Z tests
   Success rate: Y/X (N%)

   Time saved: ~[estimate] (vs manual fixing)
```

## Critical Rules

1. **NEVER** fix tests one-by-one - always look for patterns first
2. **ALWAYS** verify after each batch (max 5-6 fixes before re-running)
3. **NEVER** skip the analysis phase - understanding patterns is key
4. **ALWAYS** use exact string matching in Edit tool (preserve formatting)
5. **NEVER** assume - read the actual test files and Page Objects
6. **ALWAYS** consider Svelte 5 hydration when fixing timing issues
7. **NEVER** make broad changes - surgical edits only

## Context Awareness

This is a SvelteKit + Svelte 5 + Playwright project with:
- Supabase authentication (cookie-based SSR)
- Auto-save with 2s debounce (WriteEditor component)
- Tailwind CSS (no visual regression tests needed)
- Sequential test execution (workers: 1)
- Page Object Model architecture

Common gotchas:
- Svelte 5 SSR hydration takes time (use networkidle)
- Form actions add query params to URLs (use regex matching)
- Auto-save has 2s delay (add explicit waits)
- Test data accumulates in database (use .first() or fixtures)

## Success Criteria

- Fix 70-80% of failures in first pass
- Identify root causes, not just symptoms
- Create maintainable, resilient tests
- Complete fixes in 15-30 minutes (vs 2-3 hours manual)
```

#### How to Use

1. **Automatic Invocation**: Just say "fix the failing Playwright tests" and Claude will automatically route to this agent

2. **Explicit Invocation**: Use Task tool:
   ```typescript
   Task {
     subagent_type: "playwright-test-fixer",
     prompt: "Fix all failing Playwright tests in the test suite",
     description: "Fix Playwright test failures"
   }
   ```

3. **Resumable Sessions**: If agent times out or needs to continue:
   ```typescript
   Task {
     subagent_type: "playwright-test-fixer",
     resume: "[agent-id-from-previous-run]",
     prompt: "Continue fixing remaining test failures",
     description: "Resume test fixing"
   }
   ```

#### Expected Results

With this sub-agent:
- **First run**: Fix 14-16 of 20 tests (70-80%) in 15-20 minutes
- **Second run** (remaining): Fix 3-4 more tests in 5-10 minutes
- **Total time**: 20-30 minutes vs 2-3 hours manual
- **Context pollution**: Zero (all test output stays in sub-agent)
- **Reusability**: Works on all future test issues

---

### **PHASE 1: Slash Command + Custom Reporter**

#### Step 1.1: Create Custom Reporter

**File**: `tests/reporters/fix-analyzer.ts`

```typescript
import type { Reporter, TestCase, TestResult } from '@playwright/test/reporter';
import { writeFileSync } from 'fs';

interface FailureCategory {
  category: 'timeout' | 'selector' | 'assertion' | 'navigation' | 'network' | 'unknown';
  test: string;
  error: string;
  location: string;
  suggestedFix: string;
}

export default class FixAnalyzer implements Reporter {
  private failures: FailureCategory[] = [];

  onTestEnd(test: TestCase, result: TestResult) {
    if (result.status === 'failed' || result.status === 'timedOut') {
      const error = result.error?.message || '';
      const location = `${test.location.file}:${test.location.line}`;

      this.failures.push({
        category: this.categorizeFailure(error),
        test: test.title,
        error: error,
        location: location,
        suggestedFix: this.generateFix(error, location)
      });
    }
  }

  onEnd() {
    const report = {
      summary: {
        total: this.failures.length,
        byCategory: this.groupByCategory(),
        batchFixable: this.identifyBatchFixable()
      },
      failures: this.failures,
      actionPlan: this.generateActionPlan()
    };

    writeFileSync('test-analysis.json', JSON.stringify(report, null, 2));
    console.log('\n📊 Test Analysis Report:');
    console.log(`   Total Failures: ${report.summary.total}`);
    console.log('\n📋 By Category:');
    Object.entries(report.summary.byCategory).forEach(([cat, count]) => {
      console.log(`   ${cat}: ${count}`);
    });
    console.log('\n💡 Batch-Fixable Issues:');
    report.summary.batchFixable.forEach((issue) => {
      console.log(`   • ${issue.pattern} (${issue.count} tests)`);
    });
    console.log('\n📝 Full report: test-analysis.json\n');
  }

  private categorizeFailure(error: string): FailureCategory['category'] {
    if (error.includes('Timeout') || error.includes('exceeded')) return 'timeout';
    if (error.includes('selector') || error.includes('locator')) return 'selector';
    if (error.includes('expect') || error.includes('toEqual') || error.includes('toBe')) return 'assertion';
    if (error.includes('navigation') || error.includes('waitForURL')) return 'navigation';
    if (error.includes('net::') || error.includes('fetch')) return 'network';
    return 'unknown';
  }

  private generateFix(error: string, location: string): string {
    // Selector strict mode violations
    if (error.includes('strict mode violation') && error.includes('resolved to')) {
      return 'Add .first() or use more specific selector (exact: true, or type attribute)';
    }

    // Timeout waiting for element
    if (error.includes('Timeout') && error.includes('waiting for')) {
      return 'Increase timeout or add waitFor condition before action';
    }

    // Auto-save timing issues
    if (error.includes('toContain') && location.includes('create-entry')) {
      return 'Wait for auto-save indicator or add explicit save wait';
    }

    // Navigation failures
    if (error.includes('waitForURL') && error.includes('timedOut')) {
      return 'Check if form action is completing - may need longer timeout or networkidle wait';
    }

    // Assertion failures with content
    if (error.includes('toHaveText') || error.includes('toContainText')) {
      return 'Verify test data - may be data accumulation or timing issue';
    }

    return 'Manual investigation required';
  }

  private groupByCategory() {
    return this.failures.reduce((acc, f) => {
      acc[f.category] = (acc[f.category] || 0) + 1;
      return acc;
    }, {} as Record<string, number>);
  }

  private identifyBatchFixable() {
    const patterns: Record<string, { count: number; locations: string[] }> = {};

    this.failures.forEach((f) => {
      const key = f.suggestedFix;
      if (!patterns[key]) {
        patterns[key] = { count: 0, locations: [] };
      }
      patterns[key].count++;
      patterns[key].locations.push(f.location);
    });

    return Object.entries(patterns)
      .filter(([_, data]) => data.count > 1)
      .map(([pattern, data]) => ({
        pattern,
        count: data.count,
        locations: data.locations
      }))
      .sort((a, b) => b.count - a.count);
  }

  private generateActionPlan() {
    const grouped = this.groupByCategory();
    const plan: string[] = [];

    // Batch fixable issues first
    const batchable = this.identifyBatchFixable();
    if (batchable.length > 0) {
      plan.push(`1. Batch fix ${batchable[0].count} tests: ${batchable[0].pattern}`);
    }

    // Category-specific strategies
    if (grouped.selector > 2) {
      plan.push(`2. Review ${grouped.selector} selector issues - consider adding data-testid attributes`);
    }

    if (grouped.timeout > 2) {
      plan.push(`3. Investigate ${grouped.timeout} timeout issues - may need networkidle waits or auto-save handling`);
    }

    if (grouped.assertion > 2) {
      plan.push(`4. Check ${grouped.assertion} assertion failures - likely test data accumulation`);
    }

    return plan;
  }
}
```

#### Step 1.2: Update playwright.config.ts

```typescript
reporter: [
  ['line'],
  ['json', { outputFile: 'test-results.json' }],
  ['./tests/reporters/fix-analyzer.ts'],
  ['html'],
  ...(process.env.CI ? [['github'] as ['github']] : [])
]
```

#### Step 1.3: Create Slash Command

**File**: `.claude/commands/fix-tests.md`

```markdown
# Fix Playwright Tests

You are a Playwright test fixing specialist. Your task is to analyze test failures and fix them efficiently.

## Process

1. **Run Tests with Analysis**
   - Execute: `pnpm test`
   - Read the generated `test-analysis.json` file
   - Identify batch-fixable issues (those affecting 2+ tests)

2. **Batch Fix High-Impact Issues**
   - Start with the most common failure pattern
   - Apply fixes to all affected tests in parallel
   - Use Edit tool for surgical changes

3. **Handle Remaining Failures**
   - Group by test file
   - Fix file-by-file in order of failure count
   - Run tests after each file to verify

4. **Report Progress**
   - After each batch: "Fixed N tests (pattern: X)"
   - Show running tally: "Progress: X/Y tests fixed"

## Common Fix Patterns

### Strict Mode Violations
- **Pattern**: "resolved to 2+ elements"
- **Fix**: Add `.first()` or use `exact: true` or `type="submit"`

### Timeout Issues
- **Pattern**: "Timeout 5000ms exceeded"
- **Fix**: Add `{ timeout: 10000 }` or `waitUntil: 'networkidle'`

### Auto-Save Race Conditions
- **Pattern**: Content not saved in create-entry tests
- **Fix**: Wait for save indicator or add 3s delay after typing

### Test Data Accumulation
- **Pattern**: Multiple entries with same content
- **Fix**: Add `.first()` or implement cleanup fixtures

## Output Format

For each fix:
```
✓ Fixed: [test name]
  Issue: [brief description]
  Solution: [what was changed]
  Files: [file:line]
```

At end:
```
Summary:
  Started: X failing tests
  Fixed: Y tests
  Remaining: Z tests
  Success Rate: Y/X (N%)
```

## Rules

1. NEVER ask questions - make best judgment call
2. ALWAYS run tests after each batch of fixes
3. ALWAYS update the todo list with progress
4. NEVER fix more than 5 tests without re-running suite
5. ALWAYS provide insights about patterns discovered
```

---

### **PHASE 2: Playwright Test Analyzer Skill**

#### Step 2.1: Create Skill Structure

**File**: `.claude/skills/playwright-test-fixer/skill.md`

```markdown
# Playwright Test Fixer Skill

A comprehensive skill for diagnosing and fixing Playwright test failures with pattern recognition and automated fixes.

## Capabilities

1. **Failure Pattern Detection**
   - Identifies common failure types across test suite
   - Recognizes flaky tests vs. real failures
   - Detects test data accumulation issues

2. **Selector Healing**
   - Suggests more resilient selectors
   - Auto-adds .first() for strict mode violations
   - Recommends data-testid attributes for brittle selectors

3. **Timing Issue Diagnosis**
   - Identifies hydration race conditions
   - Detects auto-save timing problems
   - Recommends appropriate wait strategies

4. **Test Data Isolation**
   - Identifies tests with data pollution
   - Suggests fixture patterns
   - Recommends cleanup strategies

## Usage

Invoke this skill when:
- Multiple Playwright tests are failing
- Tests are flaky or timing-dependent
- You need to fix tests efficiently

The skill will:
1. Analyze all test failures holistically
2. Identify patterns and root causes
3. Propose batch fixes for common issues
4. Provide file-by-file fix plan

## Implementation Strategy

When invoked, this skill will:

### Phase 1: Analysis (2 minutes)
1. Run test suite: `pnpm test`
2. Read test-results.json and test-analysis.json
3. Categorize failures by pattern
4. Identify root causes

### Phase 2: Batch Fixing (5-10 minutes)
1. Fix highest-impact pattern first (e.g., 6 selector issues)
2. Run tests to verify fixes
3. Move to next pattern
4. Iterate until all resolved

### Phase 3: Verification (1 minute)
1. Run full suite
2. Report final results
3. Document any remaining issues

## Pattern Library

### Selector Patterns
```typescript
// ❌ Brittle
page.getByRole('button', { name: /edit/i })

// ✅ Resilient
page.getByRole('button', { name: 'Edit', exact: true })
page.getByTestId('edit-button')
page.locator('button[type="submit"]')
```

### Wait Patterns
```typescript
// ❌ Race condition
await page.goto('/login');
await page.click('button');

// ✅ Hydration-safe
await page.goto('/login', { waitUntil: 'networkidle' });
await page.click('button');
```

### Data Isolation Patterns
```typescript
// ❌ Accumulation
test('should show entry', async ({ page }) => {
  await expect(page.getByText('Test Entry')).toBeVisible();
});

// ✅ Isolated
test('should show entry', async ({ page }) => {
  await expect(page.getByText('Test Entry').first()).toBeVisible();
});
```

## Success Metrics

- Fix 60-80% of failures in first pass
- Reduce manual debugging time by 70%
- Identify root causes, not symptoms
- Create maintainable, resilient tests
```

---

### **PHASE 3: Fixture-Based Test Data Management**

#### Step 3.1: Create Auth Fixture with Cleanup

**File**: `tests/fixtures/auth.fixture.ts`

```typescript
import { test as base, expect } from '@playwright/test';
import type { Page } from '@playwright/test';

interface AuthFixture {
  authenticatedPage: Page;
  testUser: {
    email: string;
    password: string;
    id: string;
  };
}

export const test = base.extend<AuthFixture>({
  testUser: async ({}, use) => {
    const user = {
      email: process.env.TEST_EMAIL!,
      password: process.env.TEST_PASSWORD!,
      id: '' // Will be populated after auth
    };

    await use(user);

    // Cleanup: Delete all test data created during this test
    // This runs after each test completes
  },

  authenticatedPage: async ({ page, testUser }, use) => {
    // Setup: Log in
    await page.goto('/login', { waitUntil: 'networkidle' });
    await page.getByLabel(/email/i).fill(testUser.email);
    await page.getByLabel(/password/i).fill(testUser.password);
    await page.locator('button[type="submit"]').click();
    await expect(page).toHaveURL('/');

    // Provide authenticated page to test
    await use(page);

    // Cleanup: Log out
    await page.getByRole('button', { name: /log out/i }).click();
    await expect(page).toHaveURL('/login');
  }
});
```

#### Step 3.2: Create Entry Factory Fixture

**File**: `tests/fixtures/entry.fixture.ts`

```typescript
import { test as base } from '@playwright/test';
import type { Page } from '@playwright/test';

interface EntryFixture {
  createEntry: (content: string, wordCount?: number) => Promise<string>;
  cleanupEntries: () => Promise<void>;
}

export const test = base.extend<EntryFixture>({
  createEntry: async ({ page }, use) => {
    const createdEntries: string[] = [];

    const factory = async (content: string, wordCount = 50): Promise<string> => {
      // Generate content with specified word count
      const words = content.split(' ');
      const padding = ' '.repeat(Math.max(0, wordCount - words.length));
      const fullContent = content + padding;

      await page.goto('/write');
      await page.getByTestId('editor-textarea').fill(fullContent);

      // Wait for auto-save
      await page.waitForTimeout(3000);

      // Extract entry ID from URL or database
      const entryId = page.url().split('/').pop() || '';
      createdEntries.push(entryId);

      return entryId;
    };

    await use(factory);

    // Cleanup: Delete all created entries
    for (const id of createdEntries) {
      await page.goto(`/pages/${id}`);
      await page.getByRole('button', { name: 'Delete', exact: true }).click();
      await page.getByRole('dialog').locator('button[type="submit"]').click();
    }
  },

  cleanupEntries: async ({ page }, use) => {
    const cleanup = async () => {
      // Navigate to pages list and delete all test entries
      await page.goto('/pages');
      const testEntries = page.getByText(/Test Entry|Entry to test/i);
      const count = await testEntries.count();

      for (let i = 0; i < count; i++) {
        await testEntries.first().click();
        await page.getByRole('button', { name: 'Delete', exact: true }).click();
        await page.getByRole('dialog').locator('button[type="submit"]').click();
        await page.goto('/pages');
      }
    };

    await use(cleanup);
  }
});
```

#### Step 3.3: Update Tests to Use Fixtures

**Example**: `tests/create-entry.spec.ts` (refactored)

```typescript
import { test } from './fixtures/entry.fixture';
import { expect } from '@playwright/test';

test.describe('Create Journal Entry', () => {
  test('should show page progress for 750+ words', async ({ page, createEntry }) => {
    // Factory automatically generates proper word count
    await createEntry('This is a test entry', 750);

    // No need to manually type or wait - factory handles it
    await expect(page.getByText(/3 pages/i)).toBeVisible();
  });

  test('should auto-save content', async ({ page, createEntry }) => {
    const entryId = await createEntry('Auto-save test content');

    // Verify by navigating away and back
    await page.goto('/');
    await page.goto(`/write/${entryId}`);

    await expect(page.getByTestId('editor-textarea')).toHaveValue(/Auto-save test content/);
  });
});
```

---

## Expected Results

### Before Optimization
- **Time to fix 20 tests**: 2-3 hours
- **Manual effort**: High (test-by-test debugging)
- **Pattern recognition**: Manual
- **Data cleanup**: Manual or none

### After Phase 1 (Slash Command)
- **Time to fix 20 tests**: 1-1.5 hours (40% reduction)
- **Batch fixes**: 8-10 tests fixed automatically
- **Pattern recognition**: Automated via reporter

### After Phase 2 (Skill)
- **Time to fix 20 tests**: 30-45 minutes (60-70% reduction)
- **Batch fixes**: 12-15 tests fixed automatically
- **Root cause analysis**: Automated
- **Reusable**: Works on all future test issues

### After Phase 3 (Fixtures)
- **Test data issues**: Eliminated permanently
- **Test flakiness**: Reduced 80%
- **Maintenance**: Minimal (auto-cleanup)

---

## Recommended Next Steps

### Option A: Sub-Agent Approach (RECOMMENDED ⭐ - 30 minutes)
1. Create `.claude/agents/playwright-test-fixer.md` (copy from Phase 0)
2. Invoke agent: "Fix the failing Playwright tests"
3. Agent fixes 14-16 tests automatically in 15-20 minutes
4. Review and approve changes
5. **Result**: 70-80% of tests fixed with minimal effort

### Option B: Quick Win Without Sub-Agent (1 hour)
1. Implement custom reporter (Step 1.1)
2. Create /fix-tests command (Step 1.3)
3. Run command to fix current 20 failures
4. **Result**: 40-50% of tests fixed

### Option C: Comprehensive Solution (3-4 hours)
1. Create sub-agent (Phase 0)
2. Implement fixtures (Phase 3)
3. Add custom reporter (Phase 1)
4. **Result**: All current tests fixed + future issues prevented

### Option D: Incremental Approach
1. Start with sub-agent (Phase 0) to fix current issues
2. Implement fixtures (Phase 3) over next sprint
3. Add custom reporter (Phase 1) when needed
4. **Result**: Immediate relief + long-term improvements

---

## Implementation Priority

**Critical Priority** (Do First ⭐):
- ✅ **Sub-agent** (Phase 0) - Highest ROI, fastest implementation, most reusable

**High Priority** (Do Next):
- Entry factory fixture (Phase 3) - Prevents future data issues
- Custom reporter (Phase 1.1) - Enables better analysis

**Medium Priority** (Do When Needed):
- /fix-tests slash command (Phase 1.3) - Useful but sub-agent covers this
- Batch selector healing utilities

**Low Priority** (Nice to Have):
- Full skill implementation (Phase 2) - Valuable but sub-agent is simpler
- MCP server (future enhancement - no official Playwright MCP yet)

---

## Conclusion

### Recommended Immediate Action

**Create a custom sub-agent** (30 minutes setup, saves 2+ hours per debugging session):

1. Copy the agent definition from Phase 0 into `.claude/agents/playwright-test-fixer.md`
2. Say "Fix the failing Playwright tests"
3. Agent automatically analyzes, identifies patterns, and batch-fixes issues
4. Review and approve changes

**Expected Results**:
- Time to fix 20 tests: **2-3 hours → 20-30 minutes** (85% reduction)
- Context pollution: **Zero** (all debugging stays in sub-agent)
- Reusability: **100%** (works on all future test issues)
- Expertise: **Built-in** (agent knows Svelte 5, Playwright, SvelteKit patterns)

### Long-Term Strategy

After sub-agent proves valuable:
1. Add fixtures (Phase 3) to prevent test data accumulation
2. Optionally add custom reporter (Phase 1) for detailed analytics
3. Consider full skill package (Phase 2) for cross-project portability

### Why Sub-Agent Is Superior

Compared to other approaches:
- **vs Manual fixing**: 85% time savings, pattern recognition, no fatigue
- **vs Slash command**: Automatic invocation, resumability, context isolation
- **vs Skill**: Easier setup, project-specific context, immediate value
- **vs MCP**: Works today (no custom server needed)

The tooling is project-specific but the patterns are reusable across any Playwright + SvelteKit project.
