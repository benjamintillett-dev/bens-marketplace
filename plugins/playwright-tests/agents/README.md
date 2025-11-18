# Playwright Testing Plugin

**JSON-based orchestration for fixing Playwright test failures in true parallel.**

## What This Is

A Claude Code plugin that fixes failing Playwright tests with **true parallel execution**: one smart test-fixer agent per failing test (up to 20 simultaneously).

## How It Works

```
Run tests → Parse JSON → Spawn N agents → All work in parallel → Verify fixes
            (structured)  (1 per test)    (10-15 min for 20)
```

## Components

### 1. Orchestrator Agent (`orchestrator.md`)
- **Role**: Master coordinator
- **What it does**:
  1. Runs `npx playwright test --reporter=json`
  2. Parses JSON to extract failure details
  3. Spawns one test-fixer per failing test (up to 20)
  4. Verifies all fixes
  5. Reports comprehensive summary
- **Model**: Sonnet (complex planning + JSON parsing)

### 2. Test Fixer Agent (`test-fixer.md`)
- **Role**: Generic test debugger (handles ALL types of failures)
- **What it does**:
  1. Reads failing test file
  2. Analyzes error (selector? timing? assertion? auto-save?)
  3. Identifies root cause
  4. Applies appropriate fix
  5. Reports what changed and why
- **Model**: Sonnet (better quality fixes than Haiku)
- **Reusable**: Same agent handles all failure types

## How to Use

### Method 1: Slash Command (Recommended)
```
/fix-tests
```

### Method 2: Natural Language
```
"Fix my failing Playwright tests"
"I have 15 test failures, can you fix them in parallel?"
```

### Method 3: Explicit Invocation
```typescript
Task({
  subagent_type: "playwright-testing:orchestrator",
  prompt: "Fix all failing Playwright tests",
  description: "Fix tests"
});
```

## Architecture Evolution

### Old Approach (v1.0 - Category-based)
```
Orchestrator
  ├─ selector-fixer (fixes 6 tests sequentially)
  ├─ timing-optimizer (fixes 4 tests sequentially)
  ├─ autosave-handler (fixes 4 tests sequentially)
  ├─ data-cleaner (fixes 3 tests sequentially)
  └─ assertion-fixer (fixes 3 tests sequentially)

Time: 15-25 minutes for 20 tests
Agents: 5 specialists (Haiku)
Bottleneck: Each specialist works sequentially on multiple tests
```

### New Approach (v2.0 - Test-based) ✨
```
Orchestrator
  ├─ test-fixer (auth.spec.ts:34)
  ├─ test-fixer (auth.spec.ts:56)
  ├─ test-fixer (create.spec.ts:12)
  ├─ test-fixer (create.spec.ts:89)
  ├─ test-fixer (delete.spec.ts:45)
  └─ ... (up to 20 agents, all parallel)

Time: 10-15 minutes for 20 tests
Agents: 20 generic fixers (Sonnet)
Benefit: TRUE parallelism - all agents work simultaneously
```

## Expected Performance

| Scenario | Manual Fixing | With Plugin | Savings |
|----------|--------------|-------------|---------|
| 20 failing tests | 60-90 min | 10-15 min | 75-85% |
| 10 failing tests | 30-45 min | 8-12 min | 70-75% |
| 5 failing tests | 15-25 min | 5-8 min | 60-70% |

**Cost**: ~$0.03 per test (Sonnet) = ~$0.60 for 20 tests

## Example Workflow

```
You: /fix-tests

Orchestrator:
  📊 Running tests with JSON reporter...
  🔍 Found 12 failures

  Spawning 12 test-fixer agents in parallel:
    1. auth.spec.ts:34 - Timeout on button click
    2. auth.spec.ts:56 - Toggle state not updating
    3. create-entry.spec.ts:12 - waitForURL timeout
    ... [12 agents working simultaneously]

[12 agents work for 10-12 minutes]

Orchestrator:
  ✅ All agents complete!

  📈 Results:
    ✅ auth.spec.ts:34 - Fixed (added networkidle wait)
    ✅ auth.spec.ts:56 - Fixed (increased timeout)
    ✅ create-entry.spec.ts:12 - Fixed (added auto-save wait)
    ... [all 12 fixes listed]

  🎯 Final verification:
    ✓ 42 passing tests
    ✗ 0 failing tests

  ⏱️ Time: 11 minutes
  💰 Cost: $0.36
```

## What Each Test-Fixer Can Handle

The generic test-fixer agent is smart enough to handle:

### Selector Issues
- "strict mode violation" → Adds `.first()`
- "element not found" → Checks selector exists, adds wait if needed
- Multiple matches → Makes selector more specific

### Timing Issues
- Timeout on first interaction → Adds `waitForLoadState('networkidle')`
- Navigation timeout → Increases timeout or fixes redirect
- Flaky failures → Adds appropriate waits

### Auto-Save Issues (Project-Specific)
- Content not saved → Adds `waitForTimeout(3000)` for 2s debounce + network
- Knows about WriteEditor's 2-second debounce pattern

### Assertion Issues
- URL with query params → Uses `toMatch(/\/login/)` regex
- Content whitespace → Uses `toContainText()` for flexibility
- SvelteKit form actions → Handles `?/login` query params

### Data Issues
- Duplicate entries from test runs → Adds `.first()`
- Recommends cleanup fixtures for long-term solution

## Project-Specific Knowledge

This plugin knows about:
- **Svelte 5 SSR** (hydration race conditions common)
- **SvelteKit** (form action query params like `?/login`)
- **WriteEditor** (2-second auto-save debounce)
- **Supabase** (network latency considerations)
- **Sequential test execution** (workers: 1)

## Cost Breakdown

### Per-Test Cost (Sonnet)
- **Orchestrator**: ~$0.03 (runs once)
- **Test-fixer**: ~$0.03 per test

### Examples
| Tests | Orchestrator | Fixers | Total |
|-------|-------------|--------|-------|
| 5 tests | $0.03 | $0.15 | $0.18 |
| 10 tests | $0.03 | $0.30 | $0.33 |
| 20 tests | $0.03 | $0.60 | $0.63 |

**vs. Developer Time**: $50-100/hour = **100-300x ROI**

## Handling >20 Failures

If you have more than 20 failing tests:

1. **Orchestrator prioritizes** critical tests (auth, core flows)
2. **First batch**: Fixes top 20 tests in parallel
3. **Re-run**: Tests again to see what's left
4. **Second batch**: Fixes remaining tests

## File Structure

```
.claude-plugin/
└── plugin.json                 # Plugin metadata

agents/
├── README.md                   # This file
├── orchestrator.md             # Master coordinator (Sonnet)
└── test-fixer.md               # Generic test fixer (Sonnet, reusable)

commands/
└── fix-tests.md                # /fix-tests slash command
```

## vs. Other Approaches

### vs. Playwright's Official Healer Agent

| Feature | Our Plugin | Playwright Healer |
|---------|-----------|------------------|
| **Execution** | Parallel (20 at once) | Sequential (1 at a time) |
| **Speed (20 tests)** | 10-15 min | 60-90 min |
| **Approval** | Batch (trust the fixers) | Per-test (manual approval) |
| **Support** | Custom (you maintain) | Official (Playwright maintains) |
| **Trace Analysis** | Error + stack trace | Full trace (screenshots, network) |
| **Best for** | Batch fixing many tests | Creating/maintaining few tests |

### vs. Manual Fixing

| Feature | Our Plugin | Manual |
|---------|-----------|--------|
| **Speed** | 10-15 min (20 tests) | 60-90 min |
| **Consistency** | Follows patterns | Varies by developer |
| **Documentation** | Auto-generated reports | Manual notes |
| **Parallelism** | 20 agents simultaneously | 1 developer |
| **Learning** | Encoded patterns | Developer knowledge |

## Troubleshooting

### "JSON file not found"
- Ensure Playwright is installed: `npm install @playwright/test`
- Check tests exist and can run
- Try running manually: `npx playwright test`

### "Agent not spawning"
- Check agent files are in `/agents/` directory
- Verify agent name in YAML frontmatter: `name: test-fixer`

### "Fixes didn't work"
- Re-run tests to verify: `npx playwright test`
- Some fixes may uncover hidden issues
- Check test-fixer reports for details

### "Too slow"
- Ensure using Sonnet (not Opus which is slower)
- Check you're spawning agents in parallel (one message, multiple Task calls)
- Limit to 20 agents per batch

## Support

For issues or questions:
1. Check test-fixer reports for detailed fix explanations
2. Review PLAYWRIGHT_PLUGIN_ARCHITECTURE.md for design details
3. Run tests manually to verify actual state: `npx playwright test`

---

**Version**: 2.0.0 (JSON-Based Orchestration)
**Status**: Production Ready
**Last Updated**: 2025-01-18
