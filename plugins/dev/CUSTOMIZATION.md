# Customization Guide

This guide shows you how to customize the dev plugin for your specific needs.

## Quick Customizations

### Adjust Confidence Thresholds

**File:** `commands/check.md`

Change the confidence level for reported issues:

```markdown
# Current (reports issues 80+)
- Use a confidence threshold of 80+ for issues (0-100 scale)

# Stricter (reports only 90+)
- Use a confidence threshold of 90+ for issues (0-100 scale)

# More lenient (reports 70+)
- Use a confidence threshold of 70+ for issues (0-100 scale)
```

---

### Change Default Test Command

**File:** `commands/test.md`

Add your preferred test command:

```markdown
## Test Execution

Arguments provided: $ARGUMENTS

### Default Command
If no arguments provided, run: npm test -- --coverage --verbose

# Or for Python projects:
If no arguments provided, run: pytest -v --cov

# Or for Go projects:
If no arguments provided, run: go test -v ./...
```

---

### Add Project-Specific Checks

**File:** `commands/check.md`

Add custom quality checks:

```markdown
## Project-Specific Checks

In addition to standard checks, verify:
- All API endpoints have rate limiting
- Database queries use prepared statements
- Sensitive data is properly encrypted
- External API calls have timeout handling
```

---

### Customize Documentation Style

**File:** `skills/code-documentation/SKILL.md`

Adjust documentation preferences:

```markdown
## Team Preferences

Our team prefers:
- Maximum line length: 80 characters
- Always include examples for public APIs
- Use present tense ("Returns" not "Will return")
- Include type hints even when not required
```

---

## Advanced Customizations

### Add a New Command

Create a new file in `commands/`:

```bash
# Create new command
cat > commands/security-check.md << 'EOF'
---
description: Security-focused code review
allowed-tools: ["Read", "Grep", "Glob", "Bash"]
model: opus
---

# Security Check

Review code for common security vulnerabilities:

1. SQL injection risks
2. XSS vulnerabilities
3. CSRF protection
4. Authentication/authorization issues
5. Sensitive data exposure
6. Dependency vulnerabilities

$ARGUMENTS
EOF
```

Usage: `/dev:security-check`

---

### Add a New Agent

Create a new file in `agents/`:

```bash
cat > agents/performance-analyzer.md << 'EOF'
---
name: performance-analyzer
description: Analyze code for performance bottlenecks and optimization opportunities
model: sonnet
tools: ["Read", "Grep", "Glob", "Bash"]
color: purple
---

You are a performance optimization expert...

[Add detailed agent prompt here]
EOF
```

---

### Add a New Skill

Create a new directory in `skills/`:

```bash
mkdir -p skills/error-handling
cat > skills/error-handling/SKILL.md << 'EOF'
---
name: error-handling
description: Ensures proper error handling patterns are followed when writing code
allowed-tools: ["Read", "Edit"]
---

# Error Handling Skill

You are equipped with error handling best practices...

[Add detailed skill prompt here]
EOF
```

---

### Modify Agent Tool Access

**File:** `agents/bug-hunter.md`

Add or remove tools:

```yaml
# Current (read-only)
tools: ["Read", "Grep", "Glob", "Bash"]

# Add write access (use cautiously!)
tools: ["Read", "Write", "Edit", "Grep", "Glob", "Bash"]

# Minimal access
tools: ["Read", "Grep"]
```

---

### Change Agent Models

**File:** Any agent file

Adjust for cost/performance tradeoffs:

```yaml
# Current
model: opus  # Most capable, most expensive

# Change to:
model: sonnet  # Balanced (default)
model: haiku   # Fastest, cheapest
```

**Model Selection Guide:**
- **Opus**: Complex analysis (bug hunting, architecture review)
- **Sonnet**: Most tasks (refactoring, testing, documentation)
- **Haiku**: Simple, repetitive tasks (formatting, basic checks)

---

## Per-Project Customization

You can override plugin behavior on a per-project basis:

### Override a Command

Create `.claude/commands/dev/check.md` in your project with custom behavior.

### Add Project Commands

```bash
# Project-specific command
mkdir -p .claude/commands/dev
cat > .claude/commands/dev/project-check.md << 'EOF'
---
description: Project-specific quality checks
---

Check our project's specific requirements:
- Verify all components use our design system
- Check translations are complete
- Ensure accessibility standards are met
EOF
```

Usage: `/dev:project-check`

### Layer Agent Customization

```bash
# Extend bug-hunter for your project
mkdir -p .claude/agents
cat > .claude/agents/project-bug-hunter.md << 'EOF'
---
name: project-bug-hunter
description: Bug hunter with project-specific rules
model: opus
tools: ["Read", "Grep", "Glob", "Bash"]
---

You are the bug-hunter agent with additional project rules:

[Include project-specific bug patterns]
EOF
```

---

## Framework-Specific Configurations

### React Projects

Add to `commands/check.md`:

```markdown
## React-Specific Checks
- Avoid missing dependency arrays in useEffect
- Check for unnecessary re-renders
- Verify proper key props in lists
- Ensure hooks are used correctly (not in conditionals)
```

### Python Projects

Add to `commands/test.md`:

```markdown
## Python Test Configuration

Default test command:
pytest -v --cov --cov-report=term-missing --cov-fail-under=80
```

### Go Projects

Add to `commands/check.md`:

```markdown
## Go-Specific Checks
- Run go vet
- Check for goroutine leaks
- Verify proper error handling (not ignored)
- Check for race conditions with go test -race
```

---

## Environment-Specific Settings

Create a `config.json` (optional):

```json
{
  "check": {
    "confidence_threshold": 85,
    "exclude_patterns": ["vendor/", "node_modules/", "*_test.go"]
  },
  "test": {
    "default_command": "npm test",
    "timeout": 300,
    "coverage_threshold": 80
  },
  "documentation": {
    "style": "google",
    "max_line_length": 88,
    "require_examples": true
  }
}
```

Then reference in commands: `Check config.json for settings`

---

## Share Your Customizations

If you create useful customizations:

1. **Fork the repository**
2. **Add your changes**
3. **Submit a pull request** or create your own variant
4. **Share in your marketplace** for your team

---

## Tips

1. **Start Small**: Make one change at a time and test
2. **Keep Backups**: Copy original files before modifying
3. **Use Git**: Version control your plugin changes
4. **Document Changes**: Note why you made customizations
5. **Share Knowledge**: Help your team with your improvements

---

## Common Customization Patterns

### Add Language Support

If working with a language not covered:

1. Add documentation style to `skills/code-documentation/SKILL.md`
2. Add test framework detection to `commands/test.md`
3. Add language-specific checks to `commands/check.md`

### Add Company Standards

1. Create a `standards.md` reference document
2. Reference it in command prompts: "Also check against standards.md"
3. Keep standards updated as they evolve

### Integrate with CI/CD

Add hooks in `.claude/hooks/hooks.json`:

```json
{
  "pre-commit": {
    "command": "claude-code /dev:check && claude-code /dev:test"
  }
}
```

---

Need more help? Check the main README.md or Claude Code documentation.
