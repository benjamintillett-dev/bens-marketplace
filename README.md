# Ben's Development Tools - Claude Code Marketplace

A curated collection of Claude Code plugins for streamlined development workflows.

## Available Plugins

### 🛠️ dev-workflow

Comprehensive development workflow plugin with:

- **Slash Commands**: `/dev-workflow:check`, `/dev-workflow:test`, `/dev-workflow:document`
- **Specialized Agents**: bug-hunter, refactor-assistant, test-generator
- **Auto-invoked Skills**: code-documentation

**[View Plugin Documentation →](./plugins/dev-workflow/README.md)**

## Installation

### Add This Marketplace

```bash
# In Claude Code
/marketplace add https://github.com/benjamintillett-dev/dev-workflow-plugin
```

Or add via Claude Code UI: Settings → Plugins → Add Marketplace

### Install Plugins

After adding the marketplace:

```bash
/plugin install dev-workflow
```

## Plugins Included

| Plugin | Version | Description |
|--------|---------|-------------|
| dev-workflow | 1.0.0 | Code quality checks, testing assistance, documentation generation |

## Future Plugins

This marketplace will grow with additional development tools:

- Performance analysis tools
- Security scanning utilities
- Code complexity analyzers
- Documentation validators

## Repository Structure

```
dev-workflow-plugin/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace definition
├── plugins/
│   └── dev-workflow/             # Individual plugin
│       ├── .claude-plugin/
│       │   └── plugin.json       # Plugin manifest
│       ├── commands/             # Slash commands
│       ├── agents/               # Subagents
│       ├── skills/               # Skills
│       └── README.md             # Plugin docs
└── README.md                     # This file
```

## Contributing

Have suggestions or want to contribute a plugin? Feel free to open an issue or PR!

## License

MIT License - see individual plugin directories for details.
