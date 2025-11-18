# Quick Installation Guide

## Choose Your Installation Method

### 🏠 Personal Installation (Works Everywhere)

Best for individual developers who want the plugin available in all projects.

```bash
# Copy plugin to your home directory
cp -r dev ~/.claude/plugins/

# Restart Claude Code
```

**Verify installation:**
```bash
ls ~/.claude/plugins/dev
```

---

### 📦 Git Repository + Marketplace (Best for Teams)

Best for sharing with teams and easy updates.

**Step 1: Create Git Repository**

```bash
cd dev
git init
git add .
git commit -m "Initial dev plugin"

# Push to GitHub/GitLab/etc
git remote add origin https://github.com/YOURUSERNAME/dev-plugin
git push -u origin main
```

**Step 2: Create Marketplace**

```bash
# Create marketplace directory
mkdir -p ~/.claude/marketplaces

# Copy example marketplace file
cp marketplace-example.json ~/.claude/marketplaces/my-plugins.json
```

**Step 3: Edit Marketplace File**

Edit `~/.claude/marketplaces/my-plugins.json` and update the `source` URL:

```json
{
  "name": "My Development Plugins",
  "description": "Personal collection of development workflow plugins",
  "plugins": [
    {
      "name": "dev",
      "source": "https://github.com/YOURUSERNAME/dev-plugin",
      "version": "1.0.0",
      "description": "Streamlined development workflow with code quality checks, testing, and documentation"
    }
  ]
}
```

**Step 4: Install Plugin**

```bash
# In Claude Code
/plugin install dev
```

---

### 🔗 Symlink (For Plugin Development)

Best if you're actively developing and tweaking the plugin.

```bash
# Keep plugin source here for editing
cd ~/code/claude-plugins/dev

# Create symlink in home directory
ln -s ~/code/claude-plugins/dev ~/.claude/plugins/dev

# Or in a specific project
ln -s ~/code/claude-plugins/dev /path/to/project/.claude/plugins/dev
```

**Benefits:**
- Edit once, changes apply immediately
- No need to copy/update multiple times
- Perfect for iterating on commands and agents

---

### 🎯 Project-Specific Installation

Best for project-specific customization.

```bash
# Copy to your project
cp -r dev /path/to/your/project/.claude/plugins/

# Optionally commit to version control
cd /path/to/your/project
git add .claude/plugins/dev
git commit -m "Add dev plugin"
```

---

## Verify Installation

After installation with any method, restart Claude Code and verify:

```bash
# In Claude Code, try any command:
/dev:check

# Or ask Claude to use an agent:
Use the bug-hunter agent to check for issues in my recent changes
```

You should see the plugin commands available and agents responding.

---

## Next Steps

1. **Read the README**: Check out `README.md` for full feature documentation
2. **Customize**: Edit commands and agents to fit your workflow
3. **Share**: If you improve the plugin, share it with your team!

---

## Troubleshooting

**Commands not showing up?**
- Make sure you restarted Claude Code
- Check the plugin is in the right directory
- Verify file permissions (all files should be readable)

**Agents not working?**
- Check the agent frontmatter YAML is valid
- Ensure tool permissions are correct
- Try explicitly invoking with "Use the [agent-name] agent"

**Need help?**
- Check Claude Code docs: https://code.claude.com/docs
- Review example plugins: https://github.com/anthropics/claude-code/tree/main/plugins
