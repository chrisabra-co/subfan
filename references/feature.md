# Mode: Feature via competing approaches

Use when you need to *build* something and the right approach isn't obvious — multiple plausible designs, a tricky integration, an unfamiliar library, or any case where the first implementation you'd reach for might be wrong. Instead of committing to one approach and discovering its flaws late, you let several grounded approaches compete and synthesize the winner.

This is ensemble, not decomposition: every branch implements the **same feature**, end to end. They differ in *how*, not in *which part*.

## Diversity axis: distinct approaches

The branches must take genuinely different paths, or you've learned nothing. Name the axis up front. Common ways to split:

- **By design strategy** — e.g. "extend the existing abstraction" vs "add a new module" vs "inline it at the call site".
- **By priority** — "simplest thing that passes the tests" vs "most robust / handles edge cases" vs "best fit with existing patterns".
- **By library/mechanism** — when there's a real choice ("use the framework's built-in X" vs "hand-roll it" vs "pull in dependency Y").

If the feature has exactly one sane implementation, this mode is overkill — just build it.

## Grounding: each branch runs the code

A feature branch's report is only worth synthesizing if the code actually ran. Each branch MUST:

- Implement the feature for real (in an isolated worktree/copy if branches would otherwise collide on the same files — use `isolation: "worktree"` with the Workflow tool, or have each subagent work in a separate scratch copy).
- Run the build and the relevant tests, and **paste the real output** — pass/fail, error messages, exit codes.
- Note anything reality pushed back on: a test that was harder to satisfy than expected, an API that didn't behave as assumed.

If the feature has **no existing tests yet**, fix one shared acceptance test / oracle *up front* and hand the *same* one to every branch. Otherwise each branch writes its own test and grades itself against a different ruler — "convergence" then just means the branches agree on different measurements, which is meaningless.

A branch that wrote code but didn't run it is a proposal, not an experience. Weight it accordingly at the fan-in.

## Branch report contract (specialized)

```
approach        — the design strategy this branch took
what_i_did      — key implementation decisions + files touched
grounding       — build result + test output (RAW), and what fought back
result          — the diff/patch or path to the working implementation
confidence      — 0.0–1.0 it's correct & maintainable, + why
residual_risk   — edge cases not covered, tech debt incurred, untested paths
artifacts       — diff, file paths, exact test command
```

## Fan-in: pick a base, graft the rest

1. Rank branches by **grounded** confidence — a branch with passing tests and honest residual-risk beats a confident branch that didn't run anything.
2. Choose the strongest as the **base implementation**.
3. **Graft** the best ideas from runners-up: a cleaner edge-case handler here, a better name there, an error path one branch caught that the base missed. This is where ensemble beats single-shot — you assemble something better than any one branch produced. (This is still *synthesis*, not decomposition-merge: every branch built the whole feature, so you're reconciling overlapping implementations of the same thing, not stitching together disjoint pieces.)
4. On genuine disagreement about the *right* approach (not just quality), surface it to the user with the tradeoff each branch revealed, rather than silently picking. The branches just did the expensive work of de-risking each path — show that.

## Output

The synthesized implementation, plus the learnings: which approach won and *why the grounding said so*, what each losing approach taught you (the edge case it surfaced, the dead-end it ruled out), and remaining risk to watch.
