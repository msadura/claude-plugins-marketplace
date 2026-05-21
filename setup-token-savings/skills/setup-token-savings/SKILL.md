---
name: setup-token-savings
description: Use when setting up Claude Code token optimization tools, reducing API costs, installing RTK, Scrapling, Context7, LSP, code-review-graph, or configuring auto-compact and settings.json for savings
---

# Setup Token Savings

Interactive setup wizard for Claude Code token optimization. Installs and configures tools that reduce token usage by up to 86%.

## When to Use

- User wants to reduce Claude Code API costs
- User mentions RTK, Scrapling, Context7, LSP, auto-compact, token savings
- User says "save tokens", "reduce costs", "optimize budget", "setup savings"
- Starting a new machine or fresh Claude Code install

## Setup Flow

**IMPORTANT:** This skill is interactive. Ask the user before each step. Detect what's already installed and skip completed steps.

### Phase 1: Detect Current State

Run these checks silently and present a status report:

```bash
# Check installed tools
which rtk 2>/dev/null && echo "RTK: INSTALLED" || echo "RTK: NOT INSTALLED"
which scrapling 2>/dev/null && echo "Scrapling: INSTALLED" || echo "Scrapling: NOT INSTALLED"
which code-review-graph 2>/dev/null && echo "CRG: INSTALLED" || echo "CRG: NOT INSTALLED"
npx -y @upstash/context7-mcp@latest --help 2>/dev/null && echo "Context7: AVAILABLE" || echo "Context7: NEEDS NPX"

# Check settings.json for existing config
python3 -c "
import json
with open('$HOME/.claude/settings.json') as f:
    s = json.load(f)
env = s.get('env', {})
hooks = s.get('hooks', {})
print('LSP:', 'ENABLED' if env.get('ENABLE_LSP_TOOL') == '1' else 'NOT SET')
print('Auto-compact:', 'SET to ' + env.get('CLAUDE_AUTOCOMPACT_PCT_OVERRIDE', 'NOT SET'))
print('RTK Hook:', 'CONFIGURED' if any('rtk' in str(h) for h in hooks.get('PreToolUse', [])) else 'NOT SET')
print('Status line:', 'SET' if s.get('statusLine') else 'NOT SET')
"

# Check for instruction files
for f in RTK.md scrapling.md token-savings.md CLAUDE.md; do
    [ -f "$HOME/.claude/$f" ] && echo "File $f: EXISTS" || echo "File $f: MISSING"
done

# Check MCP servers
claude mcp list 2>/dev/null | grep -i "scrapling\|context7" || echo "No savings MCPs registered"
```

Present results as a checklist, then ask which steps to run.

### Phase 2: Install Tools (ask user which ones)

#### Step 1 — RTK (CLI output compression, 60-90% savings)

```bash
# Install (prefer brew on macOS)
brew install rtk-ai/tap/rtk
# OR: cargo install --git https://github.com/rtk-ai/rtk rtk

# Set up Claude Code hook (global)
rtk init -g
# NOTE: rtk init may add CLAUDE.md content — check and remove if unwanted
```

**After install, verify:**
```bash
rtk --version
# Check settings.json now has the hook
cat ~/.claude/settings.json | python3 -c "import json,sys; h=json.load(sys.stdin).get('hooks',{}); print('Hook OK' if any('rtk' in str(x) for x in h.get('PreToolUse',[])) else 'Hook MISSING')"
```

**Warn user:** `rtk init -g` may also add instructions to `~/.claude/CLAUDE.md`. Per the guide author's note, use hook-only form — extra CLAUDE.md content wastes tokens. Check and clean if needed.

#### Step 2 — Scrapling MCP (web fetch filtering, 60% savings)

```bash
# Install
pipx install "scrapling[ai]"
scrapling install

# Register as global MCP
claude mcp add --scope user ScraplingServer "$(which scrapling)" mcp
```

**Verify:**
```bash
claude mcp list 2>/dev/null | grep -i scrapling
```

Then create routing rules file:

```bash
cat > ~/.claude/scrapling.md << 'SCRAPLING_EOF'
# Scrapling MCP Server — Default Web Access

**NEVER use WebFetch when ScraplingServer is connected.**
**This applies to all contexts: main conversation, subagents, agents.**

| Task | Tool |
|------|------|
| Static page | mcp__ScraplingServer__get |
| JS-rendered SPA | mcp__ScraplingServer__fetch |
| Bot-protected (Cloudflare) | mcp__ScraplingServer__stealthy_fetch |
| GitHub | Use gh CLI |
| Documentation | Use Context7 (95% smaller) |

**Escalation:** get → fetch (if empty) → stealthy_fetch (if blocked)
SCRAPLING_EOF
```

#### Step 3 — Context7 (on-demand docs, 95% savings)

**Use the plugin** (recommended over manual MCP):
- Easier setup, auto-updates, includes usage hints teaching Claude the `resolve-library-id → get-library-docs` pipeline
- Token cost of the skill file is negligible vs 95% savings per doc lookup

Tell user to run in Claude Code session:
```
/plugin install context7
```

**Fallback** (if plugin unavailable): `claude mcp add context7 -- npx -y @upstash/context7-mcp@latest`

**Verify:** `/plugin list` should show context7, or `claude mcp list` should show it registered.

#### Step 4 — Settings.json updates (auto-compact + LSP)

Merge these into existing `~/.claude/settings.json` — do NOT overwrite existing keys:

```python
import json

with open('~/.claude/settings.json'.replace('~', __import__('os').path.expanduser('~'))) as f:
    settings = json.load(f)

# Add env vars (preserve existing)
env = settings.setdefault('env', {})
env.setdefault('ENABLE_LSP_TOOL', '1')
env.setdefault('CLAUDE_AUTOCOMPACT_PCT_OVERRIDE', '50')

# Remove effortLevel if globally set (wastes tokens)
if 'effortLevel' in settings:
    del settings['effortLevel']

with open('~/.claude/settings.json'.replace('~', __import__('os').path.expanduser('~')), 'w') as f:
    json.dump(settings, f, indent=2)
```

**IMPORTANT:** Read settings.json first, merge carefully, never clobber existing hooks/permissions.

#### Step 5 — LSP Plugins (per language)

Ask user which languages they use, then install matching LSP plugins:

| Language | Command |
|----------|---------|
| TypeScript/JS | `/plugin install typescript-lsp` |
| Python | `/plugin install pyright-lsp` |
| Go | `/plugin install gopls-lsp` |
| Rust | `/plugin install rust-analyzer-lsp` |

**Note:** LSP plugins are installed via Claude Code's `/plugin install` command — the user must run these in their Claude Code session. The skill cannot run them via Bash.

Tell user: "Run these commands in your Claude Code session: ..."

#### Step 6 — code-review-graph (optional, per-project)

```bash
pipx install code-review-graph

# Per-project setup (run in each project root):
cd <project>
code-review-graph install
code-review-graph build
```

This creates `.mcp.json` in the project — only loads for that project.

#### Step 7 — Instruction files

Create `~/.claude/token-savings.md`:

```bash
cat > ~/.claude/token-savings.md << 'SAVINGS_EOF'
# Token Savings Strategy

- ALWAYS use Context7 for docs (resolve-library-id → query-docs), NEVER raw web fetches
- NEVER use WebFetch when ScraplingServer is connected
- Tell subagents: "use Context7 to look up documentation"
- Use /compact when context window feels bloated
- Use /clear when switching to unrelated work
- Disable unused plugins per-project
SAVINGS_EOF
```

Update `~/.claude/CLAUDE.md` to reference instruction files (append, don't overwrite):

```bash
# Only add if not already referenced
grep -q "token-savings.md" ~/.claude/CLAUDE.md 2>/dev/null || cat >> ~/.claude/CLAUDE.md << 'CLAUDE_EOF'

@RTK.md
@scrapling.md
@token-savings.md
CLAUDE_EOF
```

Create `~/.claude/RTK.md`:

```bash
cat > ~/.claude/RTK.md << 'RTK_EOF'
# RTK - Rust Token Killer

**Usage**: Token-optimized CLI proxy (60-90% savings)

rtk gain          # Show token savings
rtk gain --history # Usage history
rtk discover      # Find missed opportunities

All Bash commands auto-rewritten via hook. Zero manual effort.
RTK_EOF
```

### Phase 3: Verify Everything

Run a final health check:

```bash
echo "=== Token Savings Health Check ==="
echo ""

# Tools
echo "--- Tools ---"
which rtk >/dev/null 2>&1 && echo "✓ RTK installed" || echo "✗ RTK missing"
which scrapling >/dev/null 2>&1 && echo "✓ Scrapling installed" || echo "✗ Scrapling missing"
which code-review-graph >/dev/null 2>&1 && echo "✓ code-review-graph installed" || echo "✗ code-review-graph missing (optional)"

# Settings
echo ""
echo "--- Settings ---"
python3 -c "
import json, os
with open(os.path.expanduser('~/.claude/settings.json')) as f:
    s = json.load(f)
env = s.get('env', {})
print('✓ LSP enabled' if env.get('ENABLE_LSP_TOOL') == '1' else '✗ LSP not enabled')
print('✓ Auto-compact at ' + env.get('CLAUDE_AUTOCOMPACT_PCT_OVERRIDE', '?') + '%' if 'CLAUDE_AUTOCOMPACT_PCT_OVERRIDE' in env else '✗ Auto-compact not set')
print('✗ effortLevel globally set (remove it!)' if 'effortLevel' in s else '✓ No global effortLevel (good)')
hooks = s.get('hooks', {})
print('✓ RTK hook configured' if any('rtk' in str(h) for h in hooks.get('PreToolUse', [])) else '✗ RTK hook missing')
"

# MCPs
echo ""
echo "--- MCP Servers ---"
claude mcp list 2>/dev/null | grep -iE "scrapling|context7" && echo "✓ MCPs registered" || echo "✗ No savings MCPs found"

# Files
echo ""
echo "--- Instruction Files ---"
for f in RTK.md scrapling.md token-savings.md; do
    [ -f "$HOME/.claude/$f" ] && echo "✓ $f" || echo "✗ $f missing"
done

echo ""
echo "=== Done ==="
```

### Phase 4: Tell User What to Do Manually

After automated setup, remind user:

1. **Restart Claude Code** for hooks and MCPs to take effect
2. **Install LSP plugins** in Claude Code session: `/plugin install typescript-lsp` (etc.)
3. **Audit plugins** — run `/plugin list` and disable unused ones
4. **Optional:** Install Barista statusline for monitoring (see guide)

## Quick Reference

| Tool | Install | Savings | Verify |
|------|---------|---------|--------|
| RTK | `brew install rtk-ai/tap/rtk && rtk init -g` | 60-90% CLI | `rtk gain` |
| Scrapling | `pipx install "scrapling[ai]" && scrapling install` | 60% web | `claude mcp list` |
| Context7 | `/plugin install context7` (preferred) or `claude mcp add context7 -- npx -y @upstash/context7-mcp@latest` | 95% docs | `/plugin list` |
| LSP | `ENABLE_LSP_TOOL=1` in settings.json | 95-99% nav | Check settings |
| Auto-compact | `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50` | 20-30% session | Check settings |
| CRG | `pipx install code-review-graph` | ~6.8x reviews | Per-project |

## Common Mistakes

- **Running `cargo install rtk`** installs wrong package (Rust Type Kit). Use `brew install rtk-ai/tap/rtk` or `cargo install --git https://github.com/rtk-ai/rtk rtk`
- **Setting `effortLevel: "high"` globally** wastes 10-64x tokens on simple tasks. Remove it entirely.
- **Forgetting to restart Claude Code** after changing settings.json, hooks, or MCPs
- **RTK `init -g` adding CLAUDE.md content** — check and remove extra instructions it adds; hook-only is sufficient
- **Scrapling routing conflicting with Context7** — docs should always go through Context7, not Scrapling
