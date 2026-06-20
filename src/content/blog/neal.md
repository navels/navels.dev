---
title: 'neal: a coding agent built on the loop, not the model'
description: 'neal is an open-source coding agent. The bet: quality comes from the harness — scoped planning, a coder that does not rot, and an adversarial read-only reviewer — more than from the model.'
pubDate: '2026-06-20'
---

I built [neal](https://github.com/navels/neal), an open-source coding agent that runs a planner → coder → reviewer loop from your terminal. It reads your actual source, journals every step so a killed run resumes, and lets each role use a different model.

The bet behind it: **a coding agent's quality comes mostly from the harness around the model — not the model itself.** Three ideas do the work.

## 1. Scoped planning

Before any code is written, a planner breaks the task into bounded scopes. The coder never faces "implement the whole feature" — it faces one small, well-defined piece at a time, with explicit edges on what it should and shouldn't touch. Most agent failures I've watched start as *scope* failures: the model wanders because nobody told it where the boundaries were.

## 2. A coder that doesn't rot

The failure mode of long agent runs is context rot — one ever-growing transcript where the coder accumulates noise, loses the thread, and starts contradicting its earlier self. neal runs each scope against a fresh, focused context instead of one bloated conversation. The coder stays sharp because it's never carrying the whole history — just the piece in front of it.

## 3. An adversarial, read-only reviewer

The component that decides "is this done?" should not be the component that wrote the code. neal's reviewer is **read-only by construction** — it can request changes, block, or accept, but it can never touch the code. Its job is to *falsify* the work: find the missing case, the unrequested change, the thing that doesn't match the spec. "Done" is something earned past an adversary, not self-declared.

## Does any of this actually help?

The reviewer is the one piece I could isolate cleanly, so I ran an ablation: same loop, same coder, **vary only the reviewer model** — a baseline versus a stronger one — on hard cases a single-shot coder had already failed.

| Reviewer | Cases recovered |
| --- | --- |
| Baseline | 3 / 50 |
| Stronger | 6 / 50 |

A larger (~105-case) run pointed the same way: more recoveries, and **zero regressions** — not one case the baseline reviewer passed was lost by the stronger one.

I'm **not** quoting a multiplier, because when I stress-tested it the result got humbler: neal re-samples the coder each run, and re-running cases that had passed showed ~50% flip on a fresh roll. Per-case outcomes are noisy; the *size* of the gap is uncertain. What survives is the **direction** and the **asymmetry** — a better reviewer helped, and never made a passing case fail. If it were only noise, you'd see losses too.

## The unflattering part

Reading ~90 cases neal couldn't solve, the large majority were **capability-ceiling**: the coder produced a complete, plausible patch the reviewer reasonably accepted, and it simply didn't match the hidden tests. The loop worked; the model couldn't. Which is the whole thesis restated — the model is the ceiling, and the harness is where the leverage you actually control lives: scope the work, keep the coder focused, and make it survive an adversary.

## Try it

```
npm install -g @navels/neal
```

Source and docs: [github.com/navels/neal](https://github.com/navels/neal). It runs on your existing subscription auth, keeps the reviewer read-only by construction, and resumes if you kill it mid-run.

I'd genuinely value the critical read — on the architecture and on the benchmark methodology. Tell me where it's wrong.
