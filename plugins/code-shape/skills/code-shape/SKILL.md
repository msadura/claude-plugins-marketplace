---
name: code-shape
description: Use when writing or changing code, or planning an implementation, to follow the code-shape doctrine — the rules that keep a change cheap to read: narrow interfaces, choke points, no flag threading, no dead guards, comments that state constraints instead of explaining slop, and coherent rewrites over patch-on-patch. Triggers: "add code rules", "how should I write this", "shape this change", "apply the coding doctrine", "keep this clean", or before implementing/planning a diff. This is the authoring companion to code-shape-review, which audits a diff against these same rules.
---

# Code Shape

## Overview

The doctrine for how code should be *written* and *changed* so the result is
cheap to read. Same six rules the `code-shape-review` skill audits against —
here framed as authoring guidance you apply while planning and implementing,
not after. The goal is to minimize the **cognitive cost** of a change: how much
a reader (human or AI) must hold in their head to understand it, not just how
many lines it has.

Pull this in when planning a change or before writing code. Pair it with
`code-shape-review` to check the result.

## The rules

**NARROW INTERFACE** — export the minimum. Every new export must have a
non-test external consumer in this change or the existing codebase; a "just in
case" export is a defect. Prefer module-private helpers. The wider the
interface, the more a reader must track as reachable-from-anywhere.

**CHOKE POINT** — an operation needed at N call sites must flow through ONE
point they all already pass through. Copying/inlining the logic per site is a
defect: it multiplies the diff and every future call site will forget it. When
you find yourself making the same edit twice, stop and route it through a
single point instead.

**NO FLAG THREADING** — never thread a boolean/mode parameter through multiple
layers so a distant callee behaves differently per caller. Give the owning
boundary a separate operation, or move the decision to where the information
lives. A flag threaded through three layers means three layers a reader must
trace to know what actually happens.

**NO DEAD GUARDS** — no defensive branch for a state the types or existing
callers already exclude. If you cannot construct an input that reaches a
branch, don't write it. Prefer making illegal states unrepresentable (types,
narrower parameters) over runtime checks. Every guard a reader meets is a
question ("when does this fire?"); a guard that can never fire is a question
with no answer.

**COMMENTS — THE CODE CARRIES THE MEANING, A COMMENT NEVER EXCUSES IT** — write
code a competent reader (human *and* AI) understands with NO comment. A comment
that explains, documents, motivates or **justifies** what the code does or why
it is written this way is a DEFECT — rewrite the code (rename, split,
restructure) until it reads on its own, then DELETE the comment. Three excuses
are explicitly INVALID and must never keep a comment alive: (1) *"it states the
WHY / the intent / an architectural constraint"* — justifying the shape of your
code in prose IS the slop this rule forbids; the shape must be obvious from the
code, not defended in a paragraph. (2) *"a sibling/other file in the repo is
commented like this"* — repo precedent is never a reason; copy good code, not
other files' noise. (3) *"it documents the function/export for callers"* — the
name and signature ARE the documentation; a JSDoc restating them is noise.
**Proportion is the tell:** if a unit carries comment text comparable to or
longer than its own code, the CODE is the defect and the comment is load-bearing
narration — rewrite the unit, do not annotate it. The ONLY comment that survives
states a constraint the code genuinely cannot encode and a reader cannot infer:
an external protocol quirk, a documented upstream-bug workaround, a
legal/compliance literal. "Why we chose this approach" is NOT such a constraint.

**REWRITE, DON'T PATCH** — modifying existing code means rewriting the affected
unit coherently. The result must read as if written in one pass by one author,
never as layers of appended special-cases and guards bolted onto old code. If a
change reveals the unit's design is wrong, rewrite the unit rather than wedging
the new case into it.

## Self-check before finishing

Before you consider a change done, pass each rule over your own diff:

- Did I export anything with no consumer? → make it private.
- Did I make the same edit at more than one site? → route it through one point.
- Did I add a parameter that changes a distant callee's behavior? → give the
  boundary its own operation instead.
- Did I add a branch I can't produce an input for? → delete it.
- **Did I write a comment that explains, documents or justifies the code? → the
  code is the defect; rewrite it and delete the comment. "It's the why", "a
  sibling file has one", and "it documents the export" do NOT save it. If the
  comment is as long as the code, that's the proof — rewrite. Keep only a
  constraint the code cannot encode and a reader cannot infer.**
- Does the changed unit read as one coherent pass, or as patches on patches? →
  rewrite it whole.

## Enforcing these rules in code review

These rules ship as a repo-standards file (`CODING_STANDARDS.md`) alongside this
skill. Drop that file at a repo's root and a review tool that discovers
standards docs (e.g. `mattpocock-skills:code-review`) feeds it to its Standards
axis — the review then checks the diff against each rule above by name. Use
`code-shape-review` for a self-contained audit; use this path to enforce the
same rules inside an existing two-axis (Standards + Spec) review.
