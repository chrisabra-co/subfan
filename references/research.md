# Mode: Parallel research & synthesis

Use to answer a question that benefits from multiple angles and sources — a technology comparison, a "what's the current best practice for X", a market/prior-art scan, a decision that needs grounding in what's actually out there. This is the pattern that emerges most often in practice: parallelize the *gathering of experience* (each branch reads different parts of the world), then condense.

Ensemble, not decomposition: the branches converge on the **same question**. They differ in *how they search*, not in owning separate sub-questions. (If you genuinely have separate sub-questions, that's decomposition — assign one per agent and merge; don't use this mode.)

And check the ensemble is worth it: if one agent doing a focused search would answer this well enough, just do that. Fan out only when the question is wide or contested enough that multiple independent angles genuinely cut the odds of a confident wrong answer.

## Diversity axis: different search angles

Each branch is blind to what the others find, so they must search *differently* or they'll all surface the same top-5 results. Manufacture the diversity:

- **By search modality** — one branch by official docs/specs, one by community discussion (forums, issues, Reddit/HN), one by primary data/benchmarks, one by adjacent/cross-domain analogies.
- **By stance** — one branch builds the case *for* an option, one builds the case *against*, one looks for what neither side mentions.
- **By recency/source type** — one prioritizes the most current sources, one looks for the durable/canonical references.

Name the angles before spawning. N identical "research X" prompts return correlated results and waste the fan-out.

**Diversify *sources*, not just angles.** Different angles can still funnel into the same handful of origins — one popular operator or a single curator retold across a dozen blogs *looks* like multi-source consensus but is really one voice. Tell each branch to favor sources the other angles are unlikely to reach, and consider dedicating one branch to **independent or contrarian sources** (skeptics, primary data, people with no stake in the popular narrative). You'll dedupe by `source_origin` at the fan-in regardless — but it's far cheaper to spread the sources at dispatch than to discover a monoculture afterward.

## Grounding: primary sources, quoted

Research branches ground against **sources**, and the contract must force them to bring receipts:

- Fetch and **quote** the actual source — don't paraphrase from memory.
- Capture the URL/citation for every claim so the synthesizer can re-check rather than trust.
- Flag confidence per claim: directly sourced vs inferred vs "couldn't find solid evidence".
- Note contradictions encountered, not just the tidy consensus.

A branch that returns confident claims with no citations is the research equivalent of an ungrounded verdict — discount it heavily.

## Branch report contract (specialized)

```
approach        — the search angle this branch took
what_i_did      — where it looked, what queries/sources
grounding       — KEY QUOTES + citations (URL / source) for each main claim
result          — this branch's answer to the question, from its angle
confidence      — 0.0–1.0 per major claim, + why; flag inferred vs sourced
residual_risk   — gaps, sources it couldn't access, claims it couldn't verify
artifacts       — full citation list
```

## Fan-in: corroborate, surface conflict, condense

1. **Corroboration raises confidence — but only across independent *sources*.** A claim found by branches using *different* angles is more trustworthy than one branch's finding — *provided their citations trace to different primary sources*. Angle-diversity does not guarantee source-independence: a docs branch, a forum branch, and a benchmark branch can all transitively cite the same upstream paragraph. If every angle funnels back to one origin, treat it as **single-source** (point 3) no matter how many branches surfaced it. Genuine convergence is the same claim reached from *distinct* sources — that's the trust the pattern exists to produce.
2. **Conflicting sources are a finding, not noise.** If the docs say one thing and the community says another, report *both* with their evidence — that gap is often the most useful thing the research surfaces. Don't average it into a bland middle.
3. **Single-source claims get flagged** as such, not laundered into the consensus.
4. **Completeness check:** ask what angle no branch covered. If a whole modality was missed (e.g. nobody checked primary benchmarks), that's the next branch to spawn.

## Output

A cited synthesis: the well-corroborated answer up front, conflicts and open questions called out explicitly with their evidence, confidence levels visible, and the citation trail intact so the user can verify. Condensed learnings — what the branches collectively established — not a pile of N separate summaries.
