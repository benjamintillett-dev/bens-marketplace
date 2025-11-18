# Playwright Tests Plugin

**Orchestrated multi-agent system for fixing Playwright test failures in parallel - 5x faster than manual debugging.**

## Installation

This plugin is part of the `dev-workflow` marketplace.

To use in a project:
```bash
# The plugin is already in your marketplace
# Claude Code will auto-discover it when you reference the agents
```

## Quick Start

Since custom project agents aren't automatically available as Task tool subagents, use this approach:

### Manual Orchestration (Recommended for Now)

1. Run tests: `pnpm test`
2. Analyze failures manually using agent patterns from `agents/` directory
3. Apply fixes based on specialized agent knowledge

### Future: Automatic Orchestration

When Claude Code fully supports custom agent invocation via Task tool, you'll be able to say:
```
"Fix the failing Playwright tests"
```

And Claude will automatically invoke the orchestrator.

## Plugin Structure

```
playwright-tests/
├── .claude-plugin/
│   └── plugin.json                    # Plugin metadata
├── agents/
│   ├── README.md                      # Agent usage guide
│   ├── playwright-orchestrator.md     # Coordinator (Sonnet)
│   ├── playwright-selector-fixer.md   # Selector specialist (Haiku)
│   ├── playwright-timing-optimizer.md # Timing specialist (Haiku)
│   ├── playwright-autosave-handler.md # Auto-save specialist (Haiku)
│   ├── playwright-data-cleaner.md     # Data specialist (Haiku)
│   └── playwright-assertion-fixer.md  # Assertion specialist (Haiku)
├── PLAYWRIGHT_PLUGIN_ARCHITECTURE.md  # Complete design doc
├── PLAYWRIGHT_ACCELERATION_STRATEGY.md # Research & strategy
└── README.md                          # This file
```

## Agent Knowledge Base

Each agent contains specialized knowledge for fixing specific failure types:

### 1. Selector Fixer
- Strict mode violations (`resolved to 2+ elements`)
- Add `.first()` for duplicate entries
- Use `exact: true` for button matching
- Prefer semantic locators over CSS

### 2. Timing Optimizer
- Svelte 5 SSR hydration issues
- Add `waitUntil: 'networkidle'` to Page Object goto()
- Increase timeouts for slow operations
- Wait for form action attribute updates

### 3. Auto-Save Handler
- WriteEditor 2-second debounce timing
- Add `await page.waitForTimeout(3000)` after fill()
- Handles auto-save race conditions

### 4. Data Cleaner
- Test data accumulation from multiple runs
- Add `.first()` to selectors matching duplicates
- Recommends cleanup fixtures for long-term fix

### 5. Assertion Fixer
- SvelteKit form action URL patterns (`/login?/signup`)
- Use `.toMatch(/\/login/)` instead of exact URL
- Flexible content matching with regex

## Current Workflow (Manual)

Since custom agents aren't yet invokable via Task tool:

1. **Run tests**: `pnpm test`
2. **Categorize failures** by type (selector, timing, auto-save, data, assertion)
3. **Apply fixes** using patterns from agent files as reference
4. **Verify**: Run tests again

## Future Workflow (Automatic)

When custom agent invocation is supported:

1. Say: **"Fix the failing Playwright tests"**
2. Orchestrator analyzes failures
3. Spawns 5 agents in parallel
4. Fixes complete in 15-25 minutes
5. Comprehensive summary report

## Performance Expectations

| Scenario | Manual Time | With Plugin | Savings |
|----------|-------------|-------------|---------|
| Fix 20 tests | 2-3 hours | 15-25 min | 85-90% |
| Fix 10 tests | 1-1.5 hours | 10-15 min | 80-85% |
| Fix 5 tests | 30-45 min | 5-8 min | 70-80% |

## Project-Specific Knowledge

This plugin knows about:
- **Svelte 5 SSR** (hydration race conditions)
- **SvelteKit** (form action query params)
- **WriteEditor** (2s auto-save debounce)
- **Supabase** (network latency)
- **Sequential tests** (workers: 1)

## Version History

- **1.0.0** (2025-01-18) - Week 1: Core Infrastructure
  - Orchestrator + 5 specialized sub-agents
  - Complete agent knowledge base
  - Manual workflow documented

## Future Enhancements

- **Week 2**: Auto-activating Skills for proactive debugging
- **Week 3**: Commands (`/fix-tests`, `/analyze-flaky`)
- **Week 4**: Hooks (pre-commit validation, CI integration)

## Support

For issues or questions, review:
- `PLAYWRIGHT_PLUGIN_ARCHITECTURE.md` - Complete design
- `PLAYWRIGHT_ACCELERATION_STRATEGY.md` - Research & strategy
- `agents/README.md` - Agent usage guide

---

**Marketplace**: dev-workflow
**Version**: 1.0.0
**Status**: Knowledge Base Ready (awaiting custom agent support)
**Last Updated**: 2025-01-18
