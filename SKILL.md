---
name: subfan
description: >-
  Run an ensemble fan-out — spawn N independent subagents at the SAME target, ground each against reality, then synthesize the convergent truth into condensed learnings. Use this whenever a task is high-stakes or uncertain enough that a single one-shot attempt is likely wrong: implementing a tricky feature where the approach isn't obvious, verifying whether a claim/bug/decision is actually true, or researching a question that benefits from multiple angles. Trigger on phrases like "fan out", "ensemble", "multiple approaches", "have a few agents try", "get N opinions", "verify this properly", "is this actually right", "research this deeply", "spike a few options", or any time the user wants confidence rather than a fast first guess. Prefer this over a single subagent when correctness matters more than speed. NOT for decomposition (splitting one job into disjoint subtasks) — that's a different pattern; see the gate below.
license: MIT
metadata:
  version: "0.1.0"
---

# Subfan — ensemble fan-out / fan-in

The good fan-out isn't "spray N agents to make more tokens." It's **splitting your lived experience into parallel branches, each accumulating grounded experience against reality, then re-converging with the learnings in tow.** Each branch attacks the *same target* a different way, checks its work against something real (tests, sources, attempts to refute), and reports back. The fan-in reconciles those independent experiences into one condensed, higher-confidence answer.

This skill is for **ensemble fan-out** specifically. Get this distinction right before anything else:

## Gate: is this actually an ensemble task?

There are two fan-out shapes, and they are not interchangeable:

| | **Ensemble fan-out** (this skill) | **Decomposition fan-out** (not this skill) |
|---|---|---|
| Branches attack | the **same** target, different ways | **different** disjoint subtasks |
| Fan-in does | **synthesis / consensus** — reconcile overlapping takes | **assembly / merge** — stitch non-overlapping pieces |
| Goal | **accuracy / convergence** ("is this right?") | **coverage** ("did we do everything?") |
| Example | "3 agents each verify this migration is safe" | "one agent per service, migrate each" |

If the task is decomposition — split a job into pieces that don't overlap and combine the results — **this skill is the wrong tool.** Say so plainly and just dispatch subagents normally (or use a pipeline). Don't force an ensemble onto work that has nothing to converge on.

You can only "converge" when multiple branches are looking at the same thing and can agree, disagree, or surprise each other. If there's nothing to converge on, stop here.

**Second gate — is one grounded agent enough?** Even when it *is* an ensemble task, ask whether a single agent that actually grounds (runs the test, fetches the source) would settle it. If yes, just do that. Fan out only when a lone confident answer is plausibly *and expensively* wrong — stakes and the width of the solution space justify the cost; broad phrasing ("verify this", "is this right") alone does not. Spawning 4 branches on something one agent nails is the "N for vanity" anti-pattern.

## The four principles (these are the whole game)

Everything below is mechanics. These are why it works:

1. **Manufacture diversity deliberately.** The value comes from branches exploring *different parts of reality*. Running the identical prompt N times gives correlated answers — the "convergence" is fake, you've just voted with N copies of one opinion. Force divergence: different approaches, different angles, different personas, an explicit "find a reason this is wrong." Decomposition gets diversity for free; ensemble has to engineer it. This is the failure mode that kills most ensemble fan-outs.

2. **Ground every branch against reality (back-pressure).** A branch's output is only worth synthesizing if it was checked against something real — tests it ran, code it executed, sources it fetched, an adversarial attempt to break its own claim. Back-pressure is what makes outputs converge on truth instead of on plausibility. A branch that just "thought hard" and returned prose contributes opinion, not experience.

3. **Fight lossy serialization at the fan-in.** The synthesizer does NOT remember each branch's lived experience — it reads only what each branch *serialized back*. The 20 tests a branch ran, the dead-ends it ruled out, the subtle thing it learned on attempt 14 are all lost unless the branch is forced to report them. So make every branch return a **structured contract** (below), not a freeform answer. This is the difference between a real synthesis and "summarizing summaries."

4. **Synthesize by reconciling, not averaging.** The fan-in is not a mean of N answers. It finds where branches *agree* (high confidence), where they *disagree* (the interesting part — investigate, don't split the difference), and where one branch *saw something the others missed* (graft it in). Condensed learnings, not a blended mush.

## Workflow

### Step 1 — Gate and pick the mode

Confirm it's an ensemble task (above). Then pick the mode and read its reference file for the detailed playbook:

| Mode | When | Reference |
|---|---|---|
| **Feature** | Implement something where the right approach isn't obvious — competing designs, tricky integration, "spike a few options" | `references/feature.md` |
| **Verify** | Decide whether a claim/bug/finding/decision is actually true before acting on it | `references/verify.md` |
| **Research** | Answer a question that benefits from multiple search angles and sources | `references/research.md` |

If the task spans modes (e.g. research *then* implement), run them as sequential fan-outs — read each result before designing the next. Don't nest one ensemble inside another branch.

**Diagnosis / root-cause** ("is X happening *and why*") routes to **Verify**: each branch returns its verdict *and*, if confirmed, the mechanism it reproduced — add a `cause` line to the verify contract. If the "why" is a wide-open design question rather than a bug hunt, run Verify first to confirm it's real, then a Research or Feature fan-out on the cause.

### Step 2 — Design the fan-out

Before spawning anything, **confirm each branch can actually ground** — identify the target artifact (repo path + how to run it, the source/spec URL, the reproduction steps). If it isn't in hand, get it first: locate it, or ask the user. No groundable target means this skill can't run — don't fan out into branches that can only reason (that silently breaks principle 2). Then decide and state these four things explicitly:

- **N** — how many branches. Size to stakes, not enthusiasm (see Sizing).
- **Diversity axis** — *what makes the branches different.* This is mandatory and the most important design choice. Name it: "approach A/B/C", "correctness vs security vs reproducibility lens", "search by-vendor / by-incident / by-spec". If you can't name a diversity axis, you don't have an ensemble — stop.
- **Grounding mechanism** — what real signal each branch checks against (run the test suite, execute the snippet, fetch and quote primary sources, try to refute the claim).
- **Report contract** — the structured fields each branch must return (below).

### Step 3 — Fan out

Spawn the branches **concurrently** — with the plain `Agent`/Task path that means one message with multiple tool calls; with the Workflow tool, `parallel()` (or `pipeline()`, where a later branch's grounding may begin before an earlier one finishes — also fine, see Acceleration). The point is that the branches run concurrently and independently, not one-at-a-time. Give each branch:
- The same target.
- *Its own* slice of the diversity axis ("you are taking approach B"; "you review through the security lens"; "try to refute the claim — burden of proof set by its polarity, see verify.md").
- The grounding instruction ("you MUST run the tests and paste real output").
- The report contract.

### Step 4 — Fan in

Collect the structured reports. Then synthesize per principle 4:
- **Agreements** across high-confidence branches → treat as settled.
- **Disagreements** → the signal. Investigate the conflict directly; if cheap, spawn a tie-breaker branch aimed precisely at the contested point. Never average a disagreement away.
- **Unique findings** → graft the best insight from a runner-up even if you adopt a different branch's main answer.
- Discard low-confidence, poorly-grounded branches — don't let an ungrounded vote count equally.

### Step 5 — Report condensed learnings

Deliver the converged answer plus the *learnings* — what the branches collectively established, where they disagreed and how you resolved it, and residual risk. The point of the pattern is that the experience of all branches rolls up into something the user (and you) now know with more confidence than any single attempt could provide.

## The branch report contract

Every branch returns this structure (adapt field names per mode — the references specialize it). This is principle 3 made concrete:

```
approach        — the distinct angle/lens/path THIS branch took (one line)
what_i_did      — the actual steps taken
grounding       — what real-world signal I checked against + the RAW result
                  (test output, command exit, quoted source, refutation attempt)
result          — the answer/implementation/finding this branch reached
confidence      — 0.0–1.0, AND one line on why
residual_risk   — what I could NOT verify; where this might be wrong
artifacts       — file paths, diffs, citations, commands (so the synthesizer
                  can re-check rather than trust)
```

A branch that returns `result` and `confidence` but skips `grounding` and `residual_risk` is giving you an opinion. Reject it and re-run with the contract enforced.

## Sizing

Match N to stakes and the width of the solution space, not to how fun parallelism is.

- **Quick check / narrow space:** 2–3 branches, single grounding pass.
- **Real decision / wide space:** 4–5 branches, structured synthesis, possibly a tie-breaker.
- **High-stakes (auth, payments, migrations, irreversible) / "be thorough":** a larger pool (≈6–8 branches, ≥2 of them adversarial), adversarial grounding (branches prompted to *refute*), an explicit synthesis stage, and a completeness check ("what angle did no branch cover?").

If you cap coverage (top-N, no tie-breaker, sampled sources), **say so** — silent truncation reads as "we checked everything" when you didn't.

## Acceleration: the Workflow tool (optional)

If a `Workflow` orchestration tool is available in the session (with `parallel()` / `pipeline()` and structured `schema` returns), prefer it — `schema` enforces the report contract at the tool layer (principle 3) and `pipeline()` lets branch B start verifying while branch A is still grounding. The canonical shape:

```js
const reports = await parallel(
  AXES.map(axis => () =>
    agent(branchPrompt(target, axis), { schema: BRANCH_CONTRACT })
  )
);
// reconcile reports.filter(Boolean) per principle 4 — do NOT average
```

Without the Workflow tool, the standard `Agent`/Task subagent tool does the same job — spawn the branches in one turn and reconcile their structured text replies yourself. The method is the same; only *contract enforcement* differs. Without the Workflow `schema`, the report contract is instruction-enforced rather than mechanical — so police it manually at the fan-in: reject any report missing `grounding` or `residual_risk` and re-run that branch, or principle 3 quietly fails.

## Anti-patterns

- **Fake diversity** — same prompt N times. Correlated errors, fake convergence. (Principle 1.)
- **Correlated sources** — branches with different *angles* that all trace back to the same upstream origin. Source-level fake convergence; angle-diversity isn't source-independence. Count corroboration only across *distinct* primary sources. (Principle 1, esp. research mode.)
- **Ungrounded branches** — agents that reason without checking against reality. Opinion, not experience. (Principle 2.)
- **Freeform fan-in** — branches return prose; the synthesizer summarizes summaries and the real findings evaporate. (Principle 3.)
- **Averaging disagreement** — splitting the difference between branches instead of investigating *why* they disagree. The disagreement was the most valuable signal you had. (Principle 4.)
- **Ensembling a decomposition** — forcing convergence on disjoint subtasks. Nothing to converge on; you just paid N× for a merge. (The gate.)
- **N for vanity** — 8 branches on a question that two would settle. Cost without confidence.
