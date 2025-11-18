---
description: Fix all failing Playwright tests using orchestrated parallel execution
---

# Fix Playwright Tests Command

Analyze and fix all failing Playwright tests by spawning specialized sub-agents to work in parallel.

## Process

Launch the **orchestrator** agent to coordinate the test fixing process. This agent will:

**Key Point:** The orchestrator will spawn 5 specialized sub-agents in parallel using the Task tool with multiple invocations in a single message.

The orchestrator will:

1. **Run tests** (`pnpm test`) and analyze all failures
2. **Categorize failures** by type:
   - Selector issues (strict mode violations, element not found)
   - Timing issues (timeouts, hydration race conditions)
   - Auto-save issues (2s debounce timing)
   - Data issues (test data accumulation)
   - Assertion issues (URL matching, SvelteKit form actions)

3. **Spawn 5 specialized sub-agents in parallel** (single message, multiple Task calls):
   - `playwright-testing:selector-fixer` (Haiku)
   - `playwright-testing:timing-optimizer` (Haiku)
   - `playwright-testing:autosave-handler` (Haiku)
   - `playwright-testing:data-cleaner` (Haiku)
   - `playwright-testing:assertion-fixer` (Haiku)

4. **Verify all fixes** by running tests again

5. **Report comprehensive summary** with:
   - Initial failure count by category
   - Fixes applied by each agent
   - Final pass/fail count
   - Time taken
   - Cost estimate

## Expected Performance

| Scenario | Manual Time | With Orchestration | Savings |
|----------|-------------|-------------------|---------|
| 20 failing tests | 2-3 hours | 15-25 minutes | 85-90% |
| 10 failing tests | 1-1.5 hours | 10-15 minutes | 80-85% |
| 5 failing tests | 30-45 min | 5-8 minutes | 70-80% |

## Example Usage

Simply run:
```
/fix-tests
```

The orchestrator will handle everything automatically.

## Project Context

This command knows about:
- **Svelte 5 SSR** (hydration race conditions common)
- **SvelteKit** (form action query params)
- **WriteEditor** (2-second auto-save debounce)
- **Supabase** (network latency)
- **Sequential test execution** (workers: 1)

## Success Criteria

- ✅ Fix 90%+ of tests in first orchestration pass
- ✅ Complete in 15-25 minutes (vs. 2-3 hours manual)
- ✅ No context pollution in main conversation
- ✅ Parallel efficiency: 4-5x speedup vs sequential
