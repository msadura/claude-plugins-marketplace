# Coding Standards — Code Shape

These are the repo's documented coding standards. They optimize for the
**cognitive cost** of a change — how much a reader (human or AI) must hold in
their head to understand it — not just line count. A code reviewer should cite
these rules by name when a diff violates one.

> Consumed by code review: drop this file at the repo root and a review tool
> that discovers standards docs (e.g. `mattpocock-skills:code-review`) will feed
> it to its Standards axis and check the diff against each rule below.

## Rules

### Narrow interface
Export the minimum. Every new export must have a non-test external consumer in
this change or the existing codebase; a "just in case" export is a violation.
Prefer module-private helpers. The wider the interface, the more a reader must
track as reachable-from-anywhere.

### Choke point
An operation needed at N call sites must flow through ONE point they all already
pass through. Copying or inlining the logic per site is a violation: it
multiplies the diff and every future call site will forget it. When the same
edit is made at more than one site, route it through a single point instead.

### No flag threading
Never thread a boolean/mode parameter through multiple layers so a distant
callee behaves differently per caller. Give the owning boundary a separate
operation, or move the decision to where the information lives. A flag threaded
through three layers is three layers a reader must trace to know what happens.

### No dead guards
No defensive branch for a state the types or existing callers already exclude.
If you cannot construct an input that reaches a branch, it should not exist.
Prefer making illegal states unrepresentable (types, narrower parameters) over
runtime checks. A guard that can never fire is a question a reader can't answer.

### Comments state constraints, not explanations
Code is read by humans **and** AI; write it so a capable reader needs no comment
to understand what it does or why it is shaped this way. A comment written to
make the code understandable is proof the code failed, not a fix — rewrite the
code (names, decomposition, structure) until the comment is unnecessary. Do not
add comments just because other code in the repo has them. A comment is
legitimate ONLY for a constraint the code genuinely cannot express: an external
protocol quirk, an upstream-bug workaround, a non-obvious domain invariant.
Narration, changelog/"added later" notes, and "handle edge case" markers are
violations. A long comment explaining a fragile coupling is a defect in the
coupling — restructure so the constraint holds by construction.

### Rewrite, don't patch
Modifying existing code means rewriting the affected unit coherently. The result
must read as if written in one pass by one author, never as layers of appended
special-cases and guards bolted onto old code. If a change reveals the unit's
design is wrong, rewrite the unit rather than wedging the new case into it.
