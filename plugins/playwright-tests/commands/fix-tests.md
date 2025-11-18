---
description: Fix all failing Playwright tests using JSON-based orchestration with true parallel execution
---

# Fix Playwright Tests Command

Analyze and fix all failing Playwright tests by spawning individual test-fixer agents to work in parallel.

## Process

Launch the **orchestrator** agent to coordinate the test fixing process. This agent will:

**Key Point:** The orchestrator spawns one test-fixer agent PER failing test, allowing true parallel execution (up to 20 agents simultaneously).

The orchestrator will:

1. **Run tests with JSON reporter**
   ```bash
   npx playwright test --reporter=json --output-file=test-results.json
   ```

2. **Parse structured failures** from JSON:
   - Test file and line number
   - Test name
   - Error message
   - Stack trace

3. **Spawn test-fixer agents in parallel** (one per failing test, up to 20):
   ```typescript
   Task({ subagent_type: "playwright-testing:test-fixer", model: "sonnet", ... }) // Test 1
   Task({ subagent_type: "playwright-testing:test-fixer", model: "sonnet", ... }) // Test 2
   Task({ subagent_type: "playwright-testing:test-fixer", model: "sonnet", ... }) // Test 3
   // ... up to 20 agents in a single message
   ```

4. **Each test-fixer agent**:
   - Reads the failing test file
   - Analyzes the error (selector, timing, assertion, auto-save, etc.)
   - Applies the appropriate fix
   - Reports what was changed and why

5. **Verify all fixes** by running tests again:
   ```bash
   npx playwright test --reporter=json
   ```

6. **Report comprehensive summary** with:
   - List of all fixes applied
   - Final pass/fail count
   - Success rate
   - Time taken
   - Estimated cost

## Architecture

**Old Approach** (Category-based):
- 5 specialist agents (selector-fixer, timing-optimizer, etc.)
- Each specialist fixes multiple tests sequentially

**New Approach** (Test-based):
- 1 generic test-fixer agent (reused for each test)
- One agent instance per failing test
- True parallel execution

## Expected Performance

| Scenario | Sequential Fixing | With Orchestration | Savings |
|----------|-------------------|-------------------|---------|
| 20 failing tests | 60-90 minutes | 10-15 minutes | 75-85% |
| 10 failing tests | 30-45 minutes | 8-12 minutes | 70-75% |
| 5 failing tests | 15-25 minutes | 5-8 minutes | 60-70% |

**Cost**: ~$0.03 per test fixed (Sonnet) = ~$0.60 for 20 tests

## Example Usage

Simply run:
```
/fix-tests
```

The orchestrator will handle everything automatically.

## Example Output

```
🎯 Orchestration Complete

**Initial State**: 12 failing tests
  ✅ auth.spec.ts:34 - Fixed (added networkidle wait for hydration)
  ✅ auth.spec.ts:56 - Fixed (increased timeout to 10s)
  ✅ create-entry.spec.ts:12 - Fixed (added 3s wait for auto-save debounce)
  ✅ create-entry.spec.ts:89 - Fixed (added 3s wait for auto-save debounce)
  ✅ delete-entry.spec.ts:45 - Fixed (added .first() for duplicate selector)
  ✅ delete-entry.spec.ts:67 - Fixed (added .first() for duplicate selector)
  ✅ edit-entry.spec.ts:23 - Fixed (added 3s wait for auto-save)
  ✅ view-entries.spec.ts:34 - Fixed (used regex for URL with query params)
  ✅ view-entries.spec.ts:45 - Fixed (added exact: true to button selector)
  ✅ view-entries.spec.ts:78 - Fixed (added .first() for test data pollution)
  ✅ security.spec.ts:23 - Fixed (used toContainText for flexible matching)
  ✅ security.spec.ts:56 - Fixed (added networkidle wait)

**Final State**: 12/12 tests passing ✨

**Statistics**:
  - Time: 11 minutes
  - Agents spawned: 12 (all parallel)
  - Success rate: 100%
  - Cost: ~$0.36
```

## Project Context

This command knows about:
- **Svelte 5 SSR** (hydration race conditions common)
- **SvelteKit** (form action query params)
- **WriteEditor** (2-second auto-save debounce)
- **Supabase** (network latency)
- **Test data pollution** (entries persist across runs)

## Common Fix Patterns

Each test-fixer agent can handle:

### 1. Selector Issues
- Strict mode violations → Add `.first()`
- Element not found → Verify selector, add wait if needed
- Multiple matches → Make selector more specific or use `.first()`

### 2. Timing Issues
- Timeout on first interaction → Add `waitForLoadState('networkidle')`
- Navigation timeout → Increase timeout or fix redirect
- Flaky failures → Add appropriate waits

### 3. Auto-Save Issues
- Content not saved → Add `waitForTimeout(3000)` for 2s debounce + network
- Page progress not updating → Wait for auto-save completion

### 4. Assertion Issues
- URL with query params → Use `toMatch(/\/login/)` regex
- Content whitespace → Use `toContainText()` for flexibility
- Case sensitivity → Use case-insensitive regex

### 5. Data Issues
- Duplicate entries from previous runs → Add `.first()`
- Recommend cleanup fixtures for long-term fix

## Success Criteria

- ✅ Fix 90%+ of tests in first orchestration pass
- ✅ Complete in 10-15 minutes (vs. 60-90 minutes manual)
- ✅ True parallel execution: N failing tests = N agents working simultaneously
- ✅ High-quality fixes using Sonnet (smarter than Haiku)
- ✅ Clear reporting of what was fixed and why

## Handling Many Failures

If you have more than 20 failing tests, the orchestrator will:
1. Prioritize critical tests (auth, core flows)
2. Fix top 20 in first batch
3. Re-run tests and fix remaining in second batch

## What's Different from Other Approaches

**vs. Playwright's Healer Agent:**
- ✅ Parallel execution (healer is sequential)
- ✅ Batch fixing (healer requires approval per test)
- ✅ Faster for many failures (healer is slower)
- ❌ No official support (healer is maintained by Playwright)
- ❌ Custom maintenance (healer is updated automatically)

**vs. Manual Fixing:**
- ✅ 75-85% faster
- ✅ Consistent fix patterns
- ✅ Learns project-specific patterns
- ✅ Handles 20 tests simultaneously
- ✅ Documents all changes with explanations
