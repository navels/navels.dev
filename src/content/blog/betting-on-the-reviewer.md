---
title: 'Betting a coding agent on the reviewer, not the model'
description: 'neal is an open-source coding agent built around a read-only reviewer as the gate. Here is the bet, an honest ablation, and the parts that were not flattering.'
pubDate: '2026-06-19'
---

I built [neal](https://github.com/navels/neal), an open-source coding agent that runs a **planner → coder → reviewer** loop from your terminal. It reads your actual source (no vector index), journals every step so a killed run resumes cleanly, and lets the coder and reviewer be different models.

The bet behind it is simple: **in an agent loop, the reviewer is the lever.** Most coding agents put their effort into the coder — better prompts, bigger models, more tools — and treat "is this done?" as an afterthought. neal makes the reviewer a first-class gate with one hard invariant: **it is read-only.** It can request changes, block, or accept — but it can never touch the code. The component deciding whether the work is finished has no way to declare victory by writing the answer itself.

## Does a stronger reviewer actually change outcomes?

I didn't want to assert that; I wanted to measure it. So I ran an ablation that holds everything constant except the reviewer:

- Same loop, same prompts, same harness.
- Same coder, held fixed.
- **Only the reviewer model changes** — a baseline model versus a stronger one.
- On hard cases only: tasks a single-shot coder had already failed.

Across a 50-case checkpoint:

| Reviewer | Cases recovered |
| --- | --- |
| Baseline | 3 / 50 |
| Stronger | 6 / 50 |

A larger (~105-case) continuation run pointed the same way: more recoveries, and — the number I care about most — **zero regressions**. Not one case the baseline reviewer passed was lost by the stronger one.

## The part most benchmark posts leave out

I'm deliberately **not** quoting a multiplier, because when I stress-tested the result it got humbler. neal re-samples the coder on every run, so I re-ran the cases that had passed — and **~50% of them flipped on a fresh roll.** Per-case outcomes are genuinely stochastic; the *size* of any gap is uncertain.

What survives that noise is the **direction** and the **asymmetry**. If the difference were just coder noise, you'd expect roughly as many losses as gains. Instead: gains, and zero losses. That asymmetry — a better reviewer helped, and never made a passing case fail — is the thing I'd stake the design on. The magnitude, I won't oversell.

## What I learned that wasn't flattering

- **Most failures are the model, not the harness.** Reading ~90 cases neal couldn't solve, the large majority were capability-ceiling: the coder produced a complete, plausible patch the reviewer reasonably accepted, and it simply didn't match the hidden tests. The loop worked; the model couldn't.
- **The reviewer has no oracle.** It accepts plausible-but-wrong patches because it can't see the hidden tests either. A stronger reviewer narrows that gap; it doesn't close it.
- **The model is the variable everyone argues about, and it's dwarfed by the harness around it** — how you scope work, when you stop, what's allowed to declare "done." That's where neal spends its complexity, and it's where the leverage turned out to be.

## Try it

```
npm install -g @navels/neal
```

Source and docs: [github.com/navels/neal](https://github.com/navels/neal). It runs on your existing subscription auth, keeps the reviewer read-only by construction, and resumes if you kill it mid-run.

I'd genuinely value the critical read — especially on the reviewer-as-gate idea and on the benchmark methodology. Tell me where it's wrong.
