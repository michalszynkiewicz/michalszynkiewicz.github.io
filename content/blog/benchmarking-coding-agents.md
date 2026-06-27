---
title: "How I benchmark a coding agent"
description: "A repeatable harness for measuring whether a coding agent actually got better — task suite, metrics, cost, and the failure analysis that the headline number hides."
date: 2026-06-26
draft: true   # ← scaffold for your first post. Replace the numbers, then set draft: false.
---

<div class="callout callout-draft">
  <strong>Draft scaffold.</strong> This post exists to (1) show you how code blocks,
  tables, charts, and footnotes render, and (2) give you a structure to drop your
  real benchmark into. Every number below is <em>illustrative placeholder data</em>.
  Replace it, delete this box, and flip <code>draft: false</code> in the front matter.
</div>

A demo tells you an agent *can* solve a task. A benchmark tells you how often it does, how much that costs, and whether last week's change actually helped. This is the harness I use, and how I read the results.

## The setup

Pin everything that can drift — model version, temperature, tool set, and the task suite itself — so a change in the score reflects a change you made, not noise.[^seed]

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class RunConfig:
    model: str = "claude-opus-4-8"   # REPLACE: the model under test
    temperature: float = 0.0
    max_steps: int = 40
    timeout_s: int = 600
    suite: str = "tasks/swe-lite-50.jsonl"  # REPLACE: your task set

CONFIG = RunConfig()
```

Each task runs in an isolated workspace so runs can't contaminate each other:

```bash
# REPLACE with your runner. One container per task, fixed seed, captured logs.
for task in $(jq -r '.id' "$SUITE"); do
  ./run_agent.sh --task "$task" --config config.json \
    --workspace "$(mktemp -d)" --log "logs/$task.jsonl"
done
```

## What I measure

One headline number is never enough. I track four, because they trade off against each other:

- **Resolve rate** — fraction of tasks where the agent's patch passes the hidden tests. The number everyone quotes.
- **Cost per solved task** — total token spend ÷ tasks solved. A higher resolve rate that triples cost is often the wrong trade.
- **Median steps** — how much the agent flails before converging. A proxy for wasted tokens and latency.
- **Regression rate** — tasks that passed last run and fail now. The number that actually decides whether you ship.

## Results

> The point of the table isn't the winner — it's the **cost-per-solved** column. The strongest resolve rate here also costs 2× per solved task, which may or may not be worth it for your workload.

| Variant            | Resolve rate | Cost / solved | Median steps | Regressions |
| ------------------ | -----------: | ------------: | -----------: | ----------: |
| Baseline           |       42.0 % |        $0.38  |           18 |           — |
| + retrieval        |       51.0 % |        $0.41  |           15 |           2 |
| + self-check       |       55.0 % |        $0.74  |           21 |           1 |
| + retrieval + both |     **58.0 %**|        $0.79  |           20 |           3 |

![Resolve rate vs. cost per solved task across agent variants (illustrative data).](/img/benchmark-example.svg "Resolve rate climbs with each addition, but cost per solved task climbs faster after the self-check step — exactly the trade-off the headline number hides.")

## Reading the cost

Resolve rate is the vanity metric; **cost per solved task** is the one that scales with your bill. A quick way to see where the money goes:

```python
def cost_per_solved(run):
    spend = sum(t.input_tokens * IN + t.output_tokens * OUT for t in run.tasks)
    solved = sum(1 for t in run.tasks if t.passed)
    return spend / max(solved, 1)
```

If `cost_per_solved` rises while resolve rate barely moves, the new step is buying you almost nothing — that's a cut, not a feature.

## Failure analysis

The aggregate hides the interesting part. I bucket every failure and read a sample from each bucket by hand:

1. **Wrong fix** — patch applies, tests fail. Usually a reasoning gap.
2. **No fix** — agent gives up or loops until the step cap.
3. **Broke something else** — target test passes, an unrelated one regresses.
4. **Harness fault** — flaky test, environment, timeout. *Not the agent's fault — exclude or it poisons the score.*

Bucket (4) is the one people forget, and it quietly makes every other number wrong.

## Takeaways

- A single resolve-rate number is marketing, not measurement. Report cost and regressions next to it.
- Hold the suite fixed across runs, or you're comparing two different questions.
- Read the failures. The bucket distribution tells you what to fix next far better than the headline does.

[^seed]: At `temperature = 0` you'll still see run-to-run variation from tool ordering and concurrency. Run each task 2–3× and report the median if your budget allows.
