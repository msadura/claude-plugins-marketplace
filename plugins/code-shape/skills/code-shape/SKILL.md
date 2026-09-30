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

**COMMENTS STATE CONSTRAINTS, NOT EXPLANATIONS** — code is read by humans *and*
AI; write it so a capable reader needs no comment to understand what it does or
why it is shaped this way. Reaching for an explanatory comment is a signal the
code came out unclear — the fix is to rewrite the code (better names, smaller
units, clearer structure) until the comment is unnecessary, not to annotate the
slop. Do not write comments because you saw them elsewhere in the repo; write
them only when the code genuinely cannot carry the meaning. A comment is
legitimate ONLY for a constraint the code truly cannot express: an external
protocol quirk, an upstream-bug workaround, a non-obvious domain invariant.
Narration, changelog/"added later" notes, and "handle edge case" markers are
noise — don't write them. A long comment explaining a fragile coupling is a
defect in the coupling: restructure so the constraint holds by construction.

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
- **Did I write a comment to explain what my code does? → the code is unclear;
  rewrite it so the comment isn't needed. Keep only comments that state a
  constraint the code cannot express.**
- Does the changed unit read as one coherent pass, or as patches on patches? →
  rewrite it whole.
