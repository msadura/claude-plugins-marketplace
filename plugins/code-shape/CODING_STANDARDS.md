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

### Comments — the code carries the meaning, a comment never excuses it
Write code a competent reader (human **and** AI) understands with NO comment. A
comment that explains, documents, motivates or **justifies** what the code does
or why it is written this way is a defect — rewrite the code (rename, split,
restructure) until it reads on its own, then delete the comment. Three excuses
are explicitly INVALID and a reviewer must reject them: (1) *"it states the WHY /
the intent / an architectural constraint"* — justifying the shape of your code in
prose is the slop this rule forbids; the shape must be obvious from the code.
(2) *"a sibling/other file in the repo is commented like this"* — repo precedent
is never a reason; copy good code, not other files' noise. (3) *"it documents the
function/export for callers"* — the name and signature are the documentation; a
JSDoc restating them is noise. **Proportion is the tell:** if a unit carries
comment text comparable to or longer than its own code, the code is the defect
and the comment is load-bearing narration — rewrite the unit. The only comment
that survives states a constraint the code genuinely cannot encode and a reader
cannot infer: an external protocol quirk, a documented upstream-bug workaround, a
legal/compliance literal. "Why we chose this approach" is NOT such a constraint.

### Rewrite, don't patch
Modifying existing code means rewriting the affected unit coherently. The result
must read as if written in one pass by one author, never as layers of appended
special-cases and guards bolted onto old code. If a change reveals the unit's
design is wrong, rewrite the unit rather than wedging the new case into it.
