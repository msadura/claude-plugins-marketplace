---
name: code-shape-review
description: Use when auditing a code change (branch, PR, or working tree) for cognitive-cost defects — wide module interfaces, missing choke points, boolean/mode flags threaded through layers, dead defensive branches, narration comments, patch-on-patch edits — plus diff-scoped security gaps and spec drift. Especially for AI-generated changes that pass tests and "look fine" but read as slop. Triggers: "code shape review", "review this diff/branch/PR", "is this slop", "check before I merge".
---

# Code Shape Review

## Overview

A read-only audit of a diff against a fixed rubric: six CODE SHAPE rules
(cognitive cost, not volume) + Security + Spec. A normal review catches
concrete bugs by luck of attention; this one applies each rule to the whole
diff so nothing is missed by chance, and emits a fixed output shape.

**Core principle:** judge the *shape* of the change — interface width,
reachable branches, whether an operation has one home or N copies — not just
whether it works. Code that passes tests can still be expensive to read.

## When to use

- Before merging a branch/PR, or after AI-generated changes that "look fine".
- When a diff feels like accreted patches rather than a coherent change.
- NOT for finding functional bugs first — pair with a normal correctness
  review; this rubric is about shape, security, and spec fidelity.

## Method

1. **Resolve the diff.** Default: `git diff $(git merge-base HEAD "$(git rev-parse --abbrev-ref origin/HEAD 2>/dev/null | sed 's#^origin/##' || echo main)")...HEAD`. An explicit range → that range. A PR number → `gh pr diff <n>`. No commits / dirty tree → `git diff HEAD` (working tree). Write it to one scratch file with context (`-U10`).
2. **Fresh eyes, read-only.** If this engine can dispatch a subagent (Claude `Agent`/`Task`; Codex `spawn_agent` with multi_agent; pi `pi-subagents`), dispatch ONE read-only reviewer with this rubric and the diff-file path — you are anchored on code you may have written. If no subagent tool exists, do the review inline in this session. Never edit code, never mutate the tree.
3. **Apply EVERY rule to the whole diff** (below) — one pass per rule, not a general impression. Cite `file:line` for every finding.
4. **Bounded cross-file checks only.** To confirm a NARROW INTERFACE or CHOKE POINT concern, run one focused check (grep an export's consumers, trace a threaded flag) and name both the risk and what you checked. Do not crawl the codebase.

## The rubric

The six CODE SHAPE rules below are the canonical authoring rules from the
`code-shape` skill, restated here as review criteria so this audit is
self-contained. `code-shape` is the source of truth (invoke it when writing or
planning code); this skill checks a diff against the same rules.

**CODE SHAPE** — minimize the cognitive cost of the diff:
- **NARROW INTERFACE** — every new export must have a non-test external consumer in this change or the existing codebase. A test-only or "just in case" export is a defect; prefer module-private. (Verify by grepping consumers.)
- **CHOKE POINT** — an operation needed at N call sites must flow through ONE point they all pass through. The same logic inlined/copied per site is a defect: it multiplies the diff and every future site forgets it.
- **NO FLAG THREADING** — no boolean/mode parameter threaded through multiple layers so a distant callee behaves differently per caller. Give the owning boundary a separate operation, or move the decision to where the info lives.
- **NO DEAD GUARDS** — no defensive branch for a state the types or existing callers already exclude. If you cannot construct an input that reaches it, delete it. Prefer making illegal states unrepresentable over runtime checks.
- **COMMENTS STATE CONSTRAINTS, NOT EXPLANATIONS** — code is read by humans *and* AI; a comment written to make the code understandable is proof the code failed, not a fix. Flag any comment that narrates or explains what the code does: the finding is on the *code* — rewrite it (names, decomposition, structure) so the comment is unnecessary. A comment is legitimate only for a constraint the code genuinely cannot express (protocol quirk, upstream-bug workaround, non-obvious domain invariant). Narration, changelog/"added later" notes, and "handle edge case" markers are noise. A long comment explaining a fragile coupling is a defect in the coupling — restructure so the constraint holds by construction.
- **REWRITE, DON'T PATCH** — a modified unit must read as if written in one pass by one author, never as layers of appended special-cases/guards bolted onto old code.

**SECURITY (diff-scoped)** — secrets/credentials in code or logs; injection (SQL/command/HTML/template) reachable from user or LLM input; every NEW endpoint/handler/IPC surface enforces BOTH authn AND resource ownership the way its siblings do (a missing ownership check on a new route is a finding even if authn passes); unsafe user-controlled path/URL (traversal/SSRF); unsafe deserialization/eval; over-broad file permissions; unvalidated amounts/quantities from request bodies. Any confirmed gap is Critical.

**SPEC** — the change does what it claims, no more: unrequested features, endpoints, or telemetry are scope creep (findings), not bonuses. Judge against the task/ticket/PR description if given; else against the change's own stated intent.

## Output contract

Emit exactly this shape (fixed so it is scannable and pipeline-compatible):

1. **Per-rule verdict** — one line per rule (all 6 CODE SHAPE + Security + Spec): `PASS` or `finding`.
2. **Issues** — grouped `Critical` / `Major` / `Minor`. Each: `file:line — what's wrong — one-line concrete fix`.
   - Critical = security gap, correctness break, or violated spec.
   - Major/Minor = shape violations and other non-blocking quality issues.
3. **Overall verdict** — honest one paragraph: does the change pass? Map to the dispatch dialect so manual and pipeline reviews speak alike: Critical → **BLOCK**; Major/Minor → **Simplify-targets** (fix in a cleanup pass, don't block).

Be adversarial: default to finding real violations over rubber-stamping. A stated rationale in the code ("kept simple", "follows pattern X") never downgrades a finding.

## Common mistakes

- **General impression instead of per-rule.** The value is the guaranteed pass over EACH rule — a diff that "looks clean" still gets all eight verdicts.
- **Skipping the consumer grep.** NARROW INTERFACE and CHOKE POINT need one real check, not a guess — an export with no consumer is invisible to eyeballing.
- **Editing while reviewing.** This is read-only; findings only. Fixing is a separate pass.
- **Marking shape issues Critical.** Shape defects are Major/Minor (Simplify-targets); reserve Critical for security/correctness/spec.
