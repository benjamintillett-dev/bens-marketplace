---
description: Run tests and analyze failures with actionable insights
argument-hint: "[test-pattern or test-command]"
model: sonnet
---

# Test Runner and Analyzer

Run tests and provide intelligent analysis of failures, helping you quickly understand and fix issues.

## Test Execution

Arguments provided: $ARGUMENTS

### Strategy

1. **Detect test framework** (if not specified):
   - Check for pytest, jest, vitest, rspec, go test, cargo test, etc.
   - Look at package.json, Cargo.toml, go.mod, Gemfile for clues

2. **Run appropriate test command**:
   - If argument is a command: run it directly
   - If argument is a pattern: run tests matching pattern
   - If no argument: run all tests or recently changed test files

3. **Analyze failures**:
   - Parse error messages and stack traces
   - Identify root causes (not just symptoms)
   - Check related source code for context
   - Suggest specific fixes with code examples

## Failure Analysis

For each failing test, provide:

- **Root Cause**: What's actually wrong (not just the assertion failure)
- **File Location**: Exact file:line where fix should be applied
- **Suggested Fix**: Concrete code change with reasoning
- **Related Issues**: Any other tests likely to fail for the same reason

## Success Summary

If all tests pass:
- Report total tests run and execution time
- Highlight any warnings or deprecations
- Suggest improvements to test coverage if gaps are obvious

Keep analysis focused on actionable insights, not just restating error messages.
