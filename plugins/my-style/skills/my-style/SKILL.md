---
name: my-style
description: Use when Marek asks to write, rewrite, shorten, or reply "in my style", "as me", or "BLUF" for anything that is not a PR description or PR review comment. Slack messages, Jira tickets, PRDs, specs, design notes, standup updates, emails, bug reports to other teams, replies in threads. Also use when another skill produces text he will send under his name. For PRs use pr-create-my-style; for review comments pr-review-my-style.
---

# Marek's voice

## Overview

BLUF: bottom line up front. The first line is the ask or the finding. Everything after it is
something the reader does not have yet. Real samples and before/after pairs are in `voice.md`;
read it before the first draft.

## The shape of a message

A message has these parts, in this order. Each part is one line unless it carries data.

| Part | Content |
|---|---|
| Point | The ask, the finding, or the answer. The reader could stop here |
| What the reader is missing | Only facts: error text, the input value, the URL, the curl, the command, the PR link. One line per fact, verbatim in a code block when it is error text or a command |
| Options | `a) b) c)`, max three, then `I would <pick>` with the one reason |
| Ask | One question, or the next action with a who. Nothing after it |

Drop any part that has nothing to carry. A message to a teammate who has the context is the
Point alone, often one line.

## The shape of a document

A PRD, spec, or ticket uses the sections its template demands. Each section is written as a
message above: Point first, then facts, then options. Section content is prose or `-` bullets.
The same sentence appears in one section only.

## Words

- Hedges: `imho`, `probably`, `I think`, `not sure if`, `tbh`. One per message at most.
- Flows: `canton -> evm`, `PPR + CCIPSender`, `filter + cursor`.
- Names as written in code: `sourceTransactionHash`, `/messages`, `CCIPReceiver`.
- Casual register. Lowercase starts are fine. One emoji at most, after the ask, never in a
  bug report.
- Dash is `-`. Long dashes mark generated text.
- `cc @name` on its own last line when someone else has to see it.

## Rewriting text that is not his

When the input is a draft (yours or a template), rewrite it to the shape above. The parts to
remove are known:

| In the draft | Becomes |
|---|---|
| Greeting plus "we hit a blocker" | The Point |
| `**Impact:**`, `**Context:**`, `My take:`, `Recommendation:` labels | The sentence they labelled, unlabelled |
| A sentence explaining the reader's own system or error to them | Deleted |
| A sentence that restates the previous one or the link | Deleted |
| A reason given twice | The first one |
| "Thanks!", "Keen to hear your thoughts", closing emoji | Deleted, or the Ask |
| `—` and `–` | `-` or a new sentence |

## Self-check before showing Marek

1. Line 1 is the ask or the finding.
2. Every remaining line has a value, path, error, link, or decision in it.
3. `grep -nE '—|–|\*\*[A-Za-z ]+:\*\*'` finds nothing.
4. Deleting any sentence would lose information.
5. It is shorter than the input facts, unless the input was one line.

## Common mistakes

- Treating length as respect. The CCIP API team got the repro and curls (`voice.md`, June 3);
  a teammate got `pr rebased`. Both are right.
- Explaining the regex, the endpoint, or the contract to the team that owns it.
- A tidy header per thought in a Slack message. Headers belong to documents.
- Ending a bug report with thanks instead of the ask.
