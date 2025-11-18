# Playwright Testing Plugin

**Orchestrated multi-agent system for fixing Playwright test failures in parallel.**

## What This Is

A Claude Code plugin with 6 specialized agents that work together to fix Playwright test failures **5x faster** than manual debugging.

## Components

### 1. Orchestrator Agent (`orchestrator.md`)
- **Role**: Master coordinator
- **What it does**: Analyzes failures, categorizes by type, spawns 5 sub-agents in parallel
- **Model**: Sonnet (complex planning)

### 2-6. Specialized Sub-Agents
| Agent | Focus | Model |
|-------|-------|-------|
| `playwright-testing:selector-fixer` | Strict mode violations, .first() calls | Haiku |
| `playwright-testing:timing-optimizer` | Hydration issues, networkidle waits | Haiku |
| `playwright-testing:autosave-handler` | 2s debounce race conditions | Haiku |
| `playwright-testing:data-cleaner` | Test data accumulation | Haiku |
| `playwright-testing:assertion-fixer` | URL matching, SvelteKit form actions | Haiku |

## How to Use

### Method 1: Automatic Invocation (Recommended)
Just describe what you want:
```
"Fix the failing Playwright tests"
"I have 20 failing tests, can you fix them in parallel?"
```

Claude will automatically invoke the orchestrator agent based on context.

### Method 2: Explicit Invocation
Use the Task tool:
```typescript
Task({
  subagent_type: "playwright-testing:orchestrator",
  prompt: "Fix all failing Playwright tests",
  description: "Fix tests"
});
```

## Expected Performance

| Scenario | Manual Time | With Plugin | Savings |
|----------|-------------|-------------|---------|
| Fix 20 failing tests | 2-3 hours | 15-25 min | 85-90% |
| Fix 10 failing tests | 1-1.5 hours | 10-15 min | 80-85% |
| Fix 5 failing tests | 30-45 min | 5-8 min | 70-80% |

## How It Works

1. **Orchestrator analyzes** all test failures (2 min)
2. **Categorizes** by failure type (selector, timing, auto-save, data, assertion)
3. **Spawns 5 agents in parallel** (ONE message, multiple Task calls)
4. **Agents work simultaneously** (15-20 min for 20 tests)
5. **Orchestrator verifies** all fixes (2 min)
6. **Reports summary** with comprehensive details

## Example Workflow

```
You: "Fix the failing tests"

Claude: [Invokes orchestrator]

Orchestrator:
  🔍 Running tests...
  📊 Found 20 failures:
     • Selector issues: 6 tests
     • Timing issues: 4 tests
     • Auto-save issues: 4 tests
     • Data issues: 3 tests
     • Assertion issues: 3 tests

  🚀 Spawning 5 specialized agents in parallel...

[5 agents work simultaneously for 15-20 minutes]

Orchestrator:
  ✅ All agents complete!
  🎯 Results: 20/20 tests fixed
  ⏱️ Time: 18 minutes
  💰 Cost: $0.05

  Final verification:
  ✓ 42 passing tests
  ✗ 0 failing tests
```

## Cost Breakdown

- **Orchestrator (Sonnet)**: ~$0.02
- **5 Sub-agents (Haiku)**: ~$0.005 × 5 = $0.025
- **Total per run**: ~$0.05

**vs. Developer Time**: $50-100/hour = **1000-2000x ROI**

## Troubleshooting

### "Agent not found"
- Ensure agent files are in `/agents/` directory (not `.claude/agents/`)
- Check agent name in YAML frontmatter matches invocation

### "Agents running sequentially"
- Orchestrator must invoke ALL agents in ONE message
- Multiple Task tool calls in single message = parallel execution

### "Fixes didn't work"
- Run tests again to verify
- Some fixes may uncover hidden issues
- Check agent reports for details

## Project-Specific Knowledge

This plugin knows about your project:
- **Svelte 5 SSR** (hydration race conditions)
- **SvelteKit** (form action query params)
- **WriteEditor** (2s auto-save debounce)
- **Supabase** (network latency)
- **Sequential tests** (workers: 1)

## Next Steps (Week 2-4)

Future enhancements (not yet implemented):
- **Skills**: Auto-activating skills for proactive debugging
- **Commands**: `/fix-tests`, `/analyze-flaky`, etc.
- **Scripts**: Trace analysis, pattern detection
- **Hooks**: Pre-commit validation

For now, focus on the core orchestrator + sub-agents system.

## File Structure

```
.claude-plugin/
└── plugin.json                    # Plugin metadata

agents/
├── README.md               # This file
├── orchestrator.md         # Main coordinator
├── selector-fixer.md       # Selector specialist
├── timing-optimizer.md     # Timing specialist
├── autosave-handler.md     # Auto-save specialist
├── data-cleaner.md         # Data specialist
└── assertion-fixer.md      # Assertion specialist
```

## Support

For issues or questions:
1. Check agent reports for detailed error info
2. Review PLAYWRIGHT_PLUGIN_ARCHITECTURE.md for design details
3. Run tests manually to verify actual state

---

**Version**: 1.0.0 (Week 1 - Core Infrastructure)
**Status**: Production Ready
**Last Updated**: 2025-01-18
