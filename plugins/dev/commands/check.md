---
description: Quick code quality check on recent changes
argument-hint: "[file-pattern]"
---

# Code Quality Check

Perform a focused code quality review on recent changes or specified files.

## Scope
$ARGUMENTS

## Review Focus

1. **Code Quality Issues**
   - Potential bugs or logic errors
   - Performance concerns
   - Security vulnerabilities (injection, XSS, etc.)
   - Error handling gaps

2. **Best Practices**
   - Code organization and structure
   - Naming conventions
   - DRY principle violations
   - Unnecessary complexity

3. **Maintainability**
   - Code readability
   - Comments where needed (avoid over-commenting)
   - Consistent patterns with existing codebase

## Instructions

- If no arguments provided, analyze unstaged git changes (`git diff`)
- If file pattern provided, analyze those specific files
- Use a confidence threshold of 80+ for issues (0-100 scale)
- Organize findings by severity: Critical (90-100), Important (80-89)
- Provide file:line references and specific fix suggestions
- Keep feedback actionable and concise

Focus on real issues that matter, not nitpicks. Be thorough but pragmatic.
