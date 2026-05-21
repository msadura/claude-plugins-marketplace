# Claude Plugins Marketplace

Custom plugins for Claude Code.

## Available Plugins

### setup-token-savings

Interactive setup wizard for Claude Code token optimization. Installs and configures tools that reduce token usage by up to 86%.

**Tools configured:**
- **RTK** — CLI output compression (60-90% savings)
- **Scrapling MCP** — web fetch filtering (60% savings)
- **Context7** — on-demand docs (95% savings)
- **LSP Tools** — IDE-level code intelligence (95-99% savings)
- **Auto-compact at 50%** — earlier context cleanup (20-30% savings)
- **code-review-graph** — AST-indexed code reviews (~6.8x fewer tokens)

Based on the [Claude Code Token Savings Guide](https://www.notion.so/Claude-Code-Token-Savings-Guide-Fit-a-50-mo-API-Budget-1d0e35e01f0b80b99643f0f3f56f5619).

## Installation

### 1. Add this marketplace

```
/plugin marketplace add msadura/claude-plugins-marketplace
```

### 2. Install a plugin

```
/plugin install setup-token-savings
```

### 3. Use it

Say "set up token savings" or "optimize my Claude Code budget" and the skill will guide you through the process.

## License

MIT
