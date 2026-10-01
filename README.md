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

### code-shape

Two paired skills for keeping code changes cheap to read — cognitive cost, not just line count:

- **`code-shape`** — the authoring rules, applied while planning and writing code: narrow interfaces, choke points, no flag threading, no dead guards, comments that state constraints instead of explaining slop, and coherent rewrites over patch-on-patch.
- **`code-shape-review`** — a read-only audit that checks a diff (branch, PR, or working tree) against those same rules, plus diff-scoped security gaps and spec drift. Built for AI-generated changes that pass tests and "look fine" but read as slop.

Use `code-shape` when writing or planning a change; use `code-shape-review` before merging.

**Enforce the rules inside a two-axis review (Standards + Spec):** the plugin ships `CODING_STANDARDS.md` (the code-shape rules as a repo-standards doc). Copy it to a repo's root and a review tool that discovers standards docs — e.g. [`mattpocock-skills:code-review`](https://github.com/mattpocock/skills) — feeds it to its Standards axis, so the review checks the diff against each rule by name alongside its Spec axis:

```
# copy the standards doc from the installed plugin into your repo root
# (find it with:  find ~/.claude/plugins -name CODING_STANDARDS.md -path '*code-shape*')
cp <that-path> ./CODING_STANDARDS.md
git add CODING_STANDARDS.md && git commit -m "chore: add code-shape coding standards"
# then, with mattpocock-skills installed:
/code-review <base-branch>
```

## Installation

### 1. Add this marketplace

```
/plugin marketplace add msadura/claude-plugins-marketplace
```

### 2. Install a plugin

```
/plugin install setup-token-savings
/plugin install code-shape
```

### 3. Use it

Say "set up token savings" or "optimize my Claude Code budget" and the skill will guide you through the process.

## License

MIT
