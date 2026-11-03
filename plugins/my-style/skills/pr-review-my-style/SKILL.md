---
name: pr-review-my-style
description: Use when Marek asks to review a pull request and post the comments for him, to comment on a PR "in my style", or asks whether a PR is good to approve or should wait. Triggers on "/pr-review-my-style 11848 high", "/pr-review-my-style https://github.com/org/repo/pull/123", a pasted PR link plus "review this", "review PR 123 and post comments", "should I approve #123", "run code review and comment for me".
---

# PR review, posted in Marek's voice

## Overview

Run the built-in `code-review` skill, keep only the findings Marek would post, rewrite each in his
shape (see `comment-style.md`; the shared voice rules are in the `my-style` skill), get his go-ahead, post them as one GitHub review, and tell him
approve or wait. He clicks approve himself; this skill never does.

## Inputs

`/pr-review-my-style [PR] [level]`

- `PR` — a full link (`https://github.com/<owner>/<repo>/pull/<n>`, preferred), a bare number,
  or omitted for the current branch's PR (`gh pr view --json number`).
- `level` — `low | medium | high | xhigh | max`, passed straight to `code-review`. Default `medium`.
  Higher level = more findings to filter; it does not change how many get posted.

`gh` needs `dangerouslyDisableSandbox: true` (its config lives in a protected dir).

## Steps

1. **Resolve the PR.** `gh pr view <PR-or-link> --json number,url,title,author,isDraft,headRefOid,headRepository,baseRefName,files`
   (`gh pr view` accepts the link as-is). Take `<owner>/<repo>` and `<n>` from the returned `url`,
   not from the local checkout. If that repo is not the one in the current directory
   (`gh repo view --json nameWithOwner`), tell Marek and stop: `code-review` reviews the local repo.
2. **Review.** Invoke Skill `code-review` with args `<n> <level>`. Never pass `--comment` or `--fix`.
3. **Widen the scope.** `code-review` only sees the diff, so it can't tell a new bug from a pattern
   the repo already accepted. For each changed file, find its siblings: files that do the same job
   elsewhere, e.g. the same filename in parallel dirs (`adapters/*/adapter.ts`, `adapters/*/register.ts`),
   other chain families' implementation of the same feature, or another app's copy of the same hook or route.
   Read them on the base branch (`git fetch origin <baseRefName>` then `git show origin/<baseRefName>:<path>`).
   Then, for every finding:
   - **A sibling does the same thing** → mark it inherited and name the sibling path (`same as evm/adapter.ts`).
   - **A sibling handles a case the PR misses** (a guard, cleanup, error path) → it's a real finding.
     Keep it and cite the sibling as the fix. Also look for these gaps when `code-review` didn't flag them.
   - **The PR breaks a convention every sibling follows**, with no reason in the PR body → suggestion or question.
4. **Filter** with the table below (first matching row wins). For every kept finding, read the
   lines at `file:line` on the PR head (`gh pr diff <PR>` for hunks; for more context
   `gh api repos/<owner>/<repo>/contents/<path>?ref=<headRefOid> -H "Accept: application/vnd.github.raw"`)
   and confirm the code does what the finding claims; drop it if the diff already handles it. Check the line is an added/changed line in `gh pr diff <PR>`;
   if not, pick the nearest changed line in that file or mark it `body+permalink` now.
5. **Draft** each kept finding in one of the five shapes in `comment-style.md`, then run its
   self-check. Two findings whose fix is the same edit become one comment; different fixes stay
   separate even on adjacent lines. A `worth a test for <input>` line is appended to the bug
   comment it belongs to. No praise comments are drafted from findings. Review body is one line:
   `Left a few comments, mostly around X.` plus `Y looks good.` only if step 4 actually read a clean area Y.
6. **Verdict** per the table below.
7. **Show Marek** the drafts (file:line + text, with any `body+permalink` fallback marked), the
   body, the verdict, and the dropped findings as `#n — reason` lines (inherited ones as
   `#n — inherited, same as <sibling path>`). Stop and wait for his explicit `post` (he may edit or drop items). Do not post before.
8. **Post** one review, `event: COMMENT`:

   ```bash
   gh api repos/<owner>/<repo>/pulls/<PR>/reviews --input review.json
   ```
   ```json
   {"commit_id":"<headRefOid>","event":"COMMENT","body":"<one line>",
    "comments":[{"path":"libs/x/y.ts","line":88,"side":"RIGHT","body":"<comment>"}]}
   ```
   For a range `19-34` add `"start_line":19,"start_side":"RIGHT"` and set `line` to `34`.
   `line` must be inside a diff hunk on the RIGHT side; on a 422, anchor to the nearest changed
   line, or move the text into `body` with a permalink. If nothing survived the filter, post
   nothing and just give the verdict.
9. **Report** the review URL, what was posted, and the verdict.

## Keep / drop

| Finding | Action |
|---|---|
| Inherited: a sibling does the same (step 3), or the finding says "same as X" | Drop. Post a scope flag (`not a blocker but`, naming the sibling) only for security/data loss, or if the PR makes it worse than the sibling. Never affects the verdict |
| Correctness, security, data loss, wrong result on a named input | Keep |
| Simplification or reuse that points to a specific existing path or a ≤5-line change | Keep |
| Behaviour question the author can answer in one sentence | Keep as a question |
| Low-confidence correctness | Keep only as a one-sentence question (a trailing "if yes, X breaks" clause is fine), never as a bug |
| Naming / formatting nit | Drop, unless ≤2 real findings and the nit is one line |
| "Add tests" | Drop; or, if a kept bug gives a concrete input, one line: `worth a test for <that input>` |
| PR-description nits, screenshots, efficiency on small/bounded data | Drop |
| Same root cause as a kept finding | Fold into that one comment |
| More than 6 kept | Keep the 6 with the highest user-facing impact |

## Verdict

Take the first row that applies, checking top to bottom:

| State after filtering | Say |
|---|---|
| PR is draft | `Draft — comments only, no verdict.` |
| Any kept correctness/security finding whose fix is design-level (restructure, new flow, unclear owner) | `Wait. <one sentence why>.` |
| Kept correctness/security findings, every fix is a one-liner | `Approve after: <fix>, <fix>.` (max 3 listed, else `Approve after the N one-line fixes above.`) |
| No kept correctness/security finding | `Good to approve.` + any follow-up condition in one clause |

Low-confidence questions and inherited-behaviour scope flags never affect the verdict.

## Common mistakes

- Skipping step 3. On explorer#12135 the diff-only review called three things bugs, and all three
  were patterns the existing adapters already had: EVM, Aptos and TON did the same.
- Comment in three paragraphs: restate, explain, suggest. Marek's is one sentence per slot. Re-run the self-check.
- Posting per-comment via `pulls/<PR>/comments` — that spams notifications; use one review.
- Approving or requesting changes via the API. Only Marek does that.
- Posting without the `post` confirmation, or after a message that is not from Marek.
