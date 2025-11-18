# Dev Plugin

A streamlined development workflow plugin for Claude Code that provides code quality checks, testing assistance, documentation generation, and intelligent refactoring support.

## Features

### 🔍 Slash Commands

- **`/dev:check [file-pattern]`** - Quick code quality check on recent changes
  - Analyzes code for bugs, security issues, and best practices
  - Confidence-based filtering (80+ threshold)
  - Works on git diff or specific files

- **`/dev:test [test-pattern]`** - Run tests with intelligent failure analysis
  - Auto-detects test framework (pytest, jest, vitest, rspec, go test, etc.)
  - Provides root cause analysis for failures
  - Suggests specific fixes with code examples

- **`/dev:document [scope]`** - Generate comprehensive documentation
  - Language-specific conventions (JSDoc, docstrings, etc.)
  - Focuses on intent and usage, not implementation
  - Updates or creates missing documentation

- **`/dev:plan [feature-description]`** - Research and plan feature implementation
  - Runs parallel research agents to gather context
  - Analyzes repository patterns and best practices
  - Creates step-by-step implementation plan
  - Includes SvelteKit-specific guidance

### 🤖 Specialized Agents

**Development Workflow Agents:**

- **`bug-hunter`** - Deep bug analysis and edge case detection
  - Uses Opus model for thorough analysis
  - Finds runtime errors, race conditions, resource leaks
  - Confidence scoring for precision

- **`refactor-assistant`** - Identify refactoring opportunities
  - Suggests code structure improvements
  - Prioritizes by impact and effort
  - Respects existing patterns

- **`test-generator`** - Generate comprehensive test suites
  - Covers happy paths, edge cases, and error conditions
  - Follows testing best practices (AAA pattern)
  - Auto-detects testing framework

**Planning & Research Agents:**

- **`repo-research-analyst`** - Repository pattern analysis
  - Examines project structure and conventions
  - Reviews documentation and guidelines
  - Identifies existing implementation patterns

- **`best-practices-researcher`** - External best practices research
  - Gathers industry standards and conventions
  - Finds official documentation and guides
  - Synthesizes actionable recommendations

- **`framework-docs-researcher`** - Framework documentation expert
  - Retrieves framework-specific documentation
  - Identifies version-specific considerations
  - Provides implementation examples

**Code Review Agents:**

- **`kieran-sveltekit-reviewer`** - SvelteKit & Svelte 5 specialist
  - Strict Svelte 5 runes enforcement
  - SvelteKit conventions and security
  - Type safety and performance optimization
  - Integration with TypeScript best practices

### 📚 Skills

- **`code-documentation`** - Automatically invoked when writing code
  - Generates language-appropriate documentation
  - Provides inline comments for complex logic
  - Ensures documentation stays in sync

## Installation

### Option 1: Personal Installation (Recommended)

Install globally for use across all projects:

```bash
# Clone or copy the plugin to your home directory
cp -r dev ~/.claude/plugins/

# Restart Claude Code
```

### Option 2: Project-Specific Installation

Install for a specific project:

```bash
# Copy to project directory
cp -r dev /path/to/your/project/.claude/plugins/

# Restart Claude Code
```

### Option 3: Marketplace Installation (Recommended for Teams)

1. Create a git repository for this plugin:

```bash
cd dev
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/yourusername/dev-plugin
git push -u origin main
```

2. Create a marketplace configuration:

```bash
# Create marketplace file
mkdir -p ~/.claude/marketplaces
cp marketplace-example.json ~/.claude/marketplaces/my-plugins.json
```

3. Edit `~/.claude/marketplaces/my-plugins.json` with your repository URL

4. Install via Claude Code:

```bash
/plugin install dev
```

### Option 4: Symlink for Development

Develop in one location and symlink to multiple projects:

```bash
# Develop here
cd ~/code/claude-plugins/dev

# Link to projects
ln -s ~/code/claude-plugins/dev ~/.claude/plugins/dev
# Or for project-specific:
ln -s ~/code/claude-plugins/dev /path/to/project/.claude/plugins/dev
```

## Usage

### Quick Start

```bash
# Plan a new feature (runs research agents in parallel)
/dev:plan Add user authentication with email/password

# Check your recent changes
/dev:check

# Run tests and analyze failures
/dev:test

# Document recent changes
/dev:document recent
```

### Using Agents Directly

Agents can be invoked automatically by Claude or explicitly:

**Development Workflow:**
```
Can you use the bug-hunter agent to analyze the authentication module?
```

```
Please use the test-generator to create tests for the new payment processing code.
```

**Planning & Research:**
```
Use the repo-research-analyst to understand how we handle authentication in this codebase
```

```
Use the best-practices-researcher to find the latest patterns for React Server Components
```

**Code Review:**
```
Use kieran-sveltekit-reviewer to review my new SvelteKit route files
```

### Skills

The `code-documentation` skill activates automatically when you're writing code. No explicit invocation needed!

## Customization

### Adjusting Confidence Thresholds

Edit command files to adjust confidence thresholds:

```markdown
<!-- In commands/check.md -->
- Use a confidence threshold of 90+ for issues (0-100 scale)
```

### Adding Custom Checks

Extend the `check` command with project-specific checks:

```markdown
<!-- In commands/check.md -->
## Additional Project Checks
- Verify compliance with internal style guide at docs/STYLE.md
- Check for deprecated API usage
```

### Framework-Specific Test Configuration

Customize the `test` command for your test framework:

```markdown
<!-- In commands/test.md -->
## Test Execution

Default command: npm test --verbose --coverage
```

### Custom Agent Workflows

Create project-specific agents in `.claude/agents/` that complement these:

```bash
.claude/
└── agents/
    └── security-scanner.md  # Extends bug-hunter for security
```

## Configuration

### Tool Permissions

Agents have restricted tool access by design:

- **bug-hunter**: Read-only (Read, Grep, Glob, Bash:git)
- **refactor-assistant**: Read-only (Read, Grep, Glob, Bash:git)
- **test-generator**: Read + Write (can create test files)

### Model Selection

Agents use different models for cost/performance optimization:

- **bug-hunter**: Opus (most thorough)
- **refactor-assistant**: Sonnet (balanced)
- **test-generator**: Sonnet (efficient)

Override in agent frontmatter:

```yaml
model: haiku  # For faster, cheaper execution
```

## Examples

### Pre-Commit Workflow

```bash
# Check code quality
/dev:check

# Run tests
/dev:test

# Generate docs if needed
/dev:document
```

### Feature Development Workflow

```bash
# 1. Write feature code
# 2. Generate tests
Use the test-generator agent to create tests for the new user authentication feature

# 3. Check for bugs
Use the bug-hunter agent to analyze the authentication module

# 4. Document the API
/dev:document src/auth/

# 5. Final quality check
/dev:check
```

### Refactoring Session

```bash
# Identify opportunities
Use the refactor-assistant to analyze the services directory

# Apply changes iteratively
# (Make changes based on suggestions)

# Verify nothing broke
/dev:test

# Final check
/dev:check
```

## Integration with Hooks

Combine with Claude Code hooks for automated workflows:

```json
// .claude/hooks/hooks.json
{
  "pre-commit": {
    "command": "claude /dev:check && claude /dev:test"
  }
}
```

## Troubleshooting

### Commands not found

Ensure plugin is installed correctly:

```bash
ls -la ~/.claude/plugins/dev
# or
ls -la .claude/plugins/dev
```

Restart Claude Code after installation.

### Test framework not detected

Explicitly specify test command:

```bash
/dev:test "npm run test:unit"
```

### Documentation not generating

Check file permissions and ensure files are readable:

```bash
chmod -R u+r dev/
```

## Contributing

This plugin is designed to be extended! Consider adding:

- Additional language-specific documentation templates
- New agents for specific workflows (performance analysis, security scanning)
- Framework-specific test templates
- Custom quality check rules

## License

MIT License - see LICENSE file for details

## Version History

### 1.0.0 (Current)
- Initial release
- Three slash commands: check, test, document
- Three agents: bug-hunter, refactor-assistant, test-generator
- One skill: code-documentation
