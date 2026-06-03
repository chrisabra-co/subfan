# the-subfan

A Claude skill for the **ensemble fan-out / fan-in** pattern: spawn N independent subagents at the *same* target, ground each against reality, then synthesize the convergent truth into condensed learnings.

> The good fan-out isn't "spray N agents to make more tokens." It's splitting your lived experience into parallel branches, each accumulating grounded experience against reality, then re-converging with the learnings in tow.

## Ensemble, not decomposition

There are two fan-out shapes, and they are not interchangeable:

- **Ensemble fan-out** (this skill) — branches attack the *same* target different ways; the fan-in **synthesizes** them. Goal: **accuracy / convergence** ("is this right?").
- **Decomposition fan-out** (not this skill) — branches own *different* disjoint subtasks; the fan-in **merges** them. Goal: **coverage** ("did we do everything?").

You can only converge when multiple branches look at the same thing. If there's nothing to converge on, this is the wrong tool.

## What it does

When invoked, the skill gates the task (ensemble vs decomposition), picks a mode, designs the fan-out, dispatches grounded branches in parallel, and reconciles their structured reports into one higher-confidence answer.

Three modes:

| Mode | Use for |
|---|---|
| **Feature** | Implement something where the right approach isn't obvious — competing designs compete, each grounded by running tests; the best is synthesized. |
| **Verify** | Decide whether a claim/bug/decision is *actually* true — independent, adversarial, grounded branches vote and converge. |
| **Research** | Answer a question from multiple search angles and sources, then condense into a cited synthesis. |

## The four principles

1. **Manufacture diversity deliberately** — identical prompts run N times give correlated answers and fake convergence.
2. **Ground every branch against reality** — tests, execution, primary sources, refutation attempts. Back-pressure is what makes outputs converge on truth.
3. **Fight lossy serialization at the fan-in** — branches return a *structured contract*, not prose, so the synthesizer reconciles real findings instead of summarizing summaries.
4. **Synthesize by reconciling, not averaging** — agreements settle, disagreements get investigated, unique insights get grafted.

## Portability

The method targets the universally-available subagent/Task tool, so it runs anywhere. If a `Workflow` orchestration tool is present (with `parallel()`/`pipeline()` and structured `schema` returns), the skill uses it as an accelerator — same method, better contract enforcement and latency.

## Install

```
.
├── SKILL.md            # dispatcher + shared method (the four principles, the gate, the contract)
└── references/
    ├── feature.md      # competing-approaches playbook
    ├── verify.md       # ensemble-verification playbook
    └── research.md     # parallel-research playbook
```

Drop the folder into your skills directory (e.g. `~/.claude/skills/subfan/`), or install via [skills.sh](https://skills.sh).

## License

MIT — see [LICENSE](LICENSE).
