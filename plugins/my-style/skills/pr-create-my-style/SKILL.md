---
name: pr-create-my-style
description: Use when Marek asks to create, open, or draft a pull request, write or rewrite a PR description, or fill the PR template (What / Why / Testing / Notes) "in my style". Triggers on "/pr-create-my-style", "open a PR for this branch", "write the PR description", "update the description of PR 123", "make this PR description shorter".
---

# PR description, written in Marek's style

## Overview

The code explains itself; the description tells a reviewer what changed, why, and how to see
it working. Shape: Marek's compactness + a-colleague's testing rigor. Real examples and
rewrites are in `pr-style.md`; read it before the first draft. The voice rules shared with every other
thing Marek writes (BLUF, `-` only, no labels) live in the `my-style` skill; this skill adds the PR shape.

## Inputs

`/pr-create-my-style [PR] [base]`. No PR = open a new draft PR for the current branch.
PR number or link = rewrite that PR's description. Base defaults to `develop`.
`gh` needs `dangerouslyDisableSandbox: true`.

## Steps

1. **Gather.** `git fetch origin <base>`, `git log origin/<base>..HEAD --oneline`,
   `git diff origin/<base>...HEAD --stat`, then the diff. Jira key from branch name
   (`dapp-1234`) or commits, else ask. Read the Jira ticket (Atlassian MCP) for the Why.
2. **Find the Why and the test data.** Sources: ticket, commits, this conversation. If the Why
   is not in any of them, ask Marek one question; do not invent one. Pull concrete test data
   the same way: tx hashes, message ids, addresses, preview paths, commands that were run.
3. **Draft** to the contract below.
4. **Check.** Run the checklist at the bottom on the draft.
5. **Show Marek** title + body. Wait for his OK or edits.
6. **Publish.** New PR: branch must already be on origin (never push; commits are signed by
   him). `gh pr create --draft --base <base> --title ... --body-file <tmp>`. Existing PR:
   `gh api -X PATCH repos/<owner>/<repo>/pulls/<PR> -f title=... -F body=@<tmp>` (`gh pr edit`
   fails here with a Projects (classic) GraphQL error). Then move the Jira issue to "In Review"
   (same as the `draft-pr` skill, Step 3), unless the PR is a draft stacked on another open PR;
   then ask first. Report the PR URL.

## Contract

**Title:** `[#1234] <what changed, plain words>`, under 70 characters. Number without `DAPP-`.

**Body**, these parts in this order, with a blank line after each heading:

| Part | Content | Size |
|---|---|---|
| `## What` | One bullet per change: verb + the thing in backticks + where, if not obvious. Extra detail goes in a sub-bullet. | 1-5 bullets, max 2 sub-bullets each |
| `## Why` | The problem in observable terms (error text, wrong value, who is blocked) or the constraint that forced the approach. "Dead code" is a valid Why. | 1-3 bullets |
| `## Testing` | Line 1: what automated tests cover, or `No automated tests: <reason>`. Then manual: where (preview deployment / local command), each action as a bullet, each expected result as a `Confirm ...` sub-bullet, with the real hash / address / link / command. | Always present |
| `## Notes` | Only: merge order or stacked-on PR; diff-size breakdown when lockfile or generated code is most of the diff; deliberate omission or known gap a reviewer would ask about; env vars or setup needed; follow-up ticket. | Omit the section when none apply |
| footer | `[JIRA](https://<jira-host>/browse/DAPP-1234)` | Last line |

**Length budget** (body lines, counting from `## What` to the footer; diff size excludes lockfile and generated files):

| Diff | Body |
|---|---|
| up to 20 changed lines | 12 lines max |
| up to 300 | 25 lines max |
| larger | 40 lines max |

## Checklist before showing Marek

1. `grep -nE '—|–'` on the draft finds nothing. The only dash is `-`.
2. No template placeholder text left ("Why is this change needed?", "Notes are optional").
3. Within the length budget. If over, cut What sub-bullets first, then Notes.
4. Every Testing step names something concrete: a link, hash, address, command, or path.
5. Every Notes bullet fits one of the five Notes kinds. Review findings about the code are not Notes; tell Marek separately.
6. Nothing in What restates logic the diff already shows (field lists, sort orders, per-assertion test lists).
