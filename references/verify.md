# Mode: Ensemble verification

Use when you need to know whether something is **actually true** before acting on it — a reported bug, a security finding, a "this migration is safe" claim, an architectural decision, a benchmark result. This is the core pattern: independent branches stress the same claim against reality and you converge on a verdict instead of trusting one plausible-sounding answer.

The whole point is to resist plausibility. A single agent will often produce a confident, well-written, *wrong* verdict. Independent grounded branches that are actively trying to break the claim catch what one pass misses.

**First, check an ensemble is even warranted.** If a single grounded agent — one that actually runs the test, executes the query, or reads the source — would settle this, just do that. Fan out only when a lone confident answer is plausibly *and expensively* wrong (high stakes, or a claim that's easy to get convincingly wrong).

## Diversity axis: independent + adversarial lenses

Two complementary ways to manufacture diversity — use whichever fits, or both:

- **Adversarial (vote to refute):** spawn N independent skeptics, each instructed to *try to refute* the claim. **Define the claim and its polarity explicitly first**, because the burden of proof flips:
  - *Existence / bug claims* ("this bug is real", "this query is slow") — default to `refuted` unless the branch can positively **reproduce** it. The claim survives only if a majority reproduce it.
  - *Absence / safety claims* ("no data race", "no leak", "the migration is safe") — you **cannot positively prove absence**, so the branch's job is to hunt a **counterexample**. "No counterexample found after stated effort X" is a *pass* signal, not a refutation; if the effort was shallow, return `inconclusive`, never `refuted`. The claim survives only if no branch produces a counterexample.

  Either way the bias is against false confidence, which is the expensive error. (Getting this backwards — letting a skeptic "refute" a safety claim just because they couldn't prove the absence — systematically votes down true safety claims. Don't.)
- **Perspective-diverse:** when a claim can fail in more than one way, give each branch a distinct lens — *correctness*, *security*, *does-it-actually-reproduce*, *performance*, *data-integrity*. Diversity of failure mode catches what N identical skeptics can't.

Identical "please verify this" prompts run N times do **not** count — that's fake convergence. The branches must differ in stance or lens.

## Grounding: reproduce, don't reason

A verification branch must check the claim against reality, not just think about it:

- **Reproduce it** — run the failing case, execute the query, trigger the code path, and paste raw output.
- **Re-derive from source** — read the actual code/data/spec rather than trusting the claim's description of it.
- **Construct the counterexample** — for a "this is safe/always works" claim, actively search for the input that breaks it.

**Irreversible / non-reproducible claims** (production migrations, one-way actions): running the action *is* the irreversible event you're trying to de-risk, so "reproduce it" is off the table. Ground instead by reproducing on a clone / staging / shadow environment, or by re-deriving from source plus constructing a counterexample. Here, re-derivation counts as real grounding (don't discard those branches at the fan-in), and "inconclusive — cannot safely reproduce" is a legitimate, honest verdict rather than a failure.

If a branch writes a repro script or a failing test that could collide with sibling branches in the same repo, isolate it (a separate worktree / scratch copy — see `feature.md`).

A branch that returns "looks correct to me" with no reproduction *and no re-derivation* is noise. Reject ungrounded verdicts at the fan-in.

## Branch report contract (specialized)

```
approach        — the lens/stance this branch took (e.g. "refute via security")
what_i_did      — how I tried to confirm or break the claim
grounding       — reproduction steps + RAW result (output, exit code, quoted source)
result          — verdict: confirmed | refuted | inconclusive
confidence      — 0.0–1.0 in the verdict, + why
residual_risk   — what I couldn't test; conditions under which my verdict flips
artifacts       — commands, file:line references, the counterexample if found
```

## Fan-in: weighted consensus, investigate dissent

1. **Tally grounded verdicts.** Discard branches that neither reproduced the claim *nor* re-derived it from primary source — an ungrounded vote doesn't count equally. (For irreversible claims where reproduction is unsafe, re-derivation from source *is* grounding; don't discard those.)
2. **Majority of grounded branches** sets the provisional verdict — for an adversarial axis, the claim is *confirmed* only if a majority failed to refute it (per the polarity rule above); otherwise refuted or inconclusive. When adversarial and perspective-diverse branches are mixed, weight by grounding quality, not raw head-count. For high-stakes claims, require a real majority (e.g. ≥2 of 3, ≥3 of 5), not a plurality.
3. **Disagreement is the payload — investigate it before resolving.** If branches split, one of them found something the others didn't. Read the dissenter's `grounding` and `artifacts` directly first — often the dissenter is right and the majority was sloppy. Only if it's still unresolved and cheap, spawn a tie-breaker aimed exactly at the contested point.
4. Never report a verdict more confident than the grounding supports. "Inconclusive — couldn't reproduce" is a valid, honest output.

## Output

The verdict, the vote breakdown, the strongest piece of grounding behind it (the reproduction or the counterexample), how any dissent was resolved, and the conditions under which the verdict would change. If high-stakes and unverifiable, say *that* clearly rather than manufacturing a confident answer.
