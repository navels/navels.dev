---
title: 'neal: a coding agent that coordinates multiple models'
description: 'neal is an open-source coding agent that puts a different model in each seat — planner, coder, reviewer — and coordinates them in a scoped loop. The bet: coordination beats reaching for one bigger model.'
pubDate: '2026-06-20'
---

I built [neal](https://github.com/navels/neal), an open-source coding agent that **coordinates multiple models** — a planner, a coder, and a reviewer, each its own model — in a scoped loop you run from your terminal. It reads your actual source, journals every step so a killed run resumes, and lets you put a different model in each seat.

The bet behind it: **how you coordinate models around a task beats reaching for one bigger model.** Three pieces make the coordination work.

## 1. A planner that scopes the work

Before any code is written, a planner breaks the task into bounded scopes. The coder never faces "implement the whole feature" — it faces one small, well-defined piece with explicit edges on what it should and shouldn't touch. Most agent failures I've watched start as *scope* failures: the model wanders because nobody drew the boundaries.

## 2. A coder that stays focused

The failure mode of long agent runs is context rot — one ever-growing transcript where the coder accumulates noise, loses the thread, and contradicts its earlier self. neal runs each scope against a fresh, focused context. The coder stays sharp because it's only ever carrying the piece in front of it — not the whole history.

## 3. A *different* model as the reviewer

The model that decides "is this done?" should not be the model that wrote the code — it shares its own blind spots. neal's reviewer is **a separate model, read-only by construction**: it can request changes, block, or accept, but it can never touch the code. A different model reviewing makes the check genuinely adversarial — it catches the missing case, the unrequested change, the thing that doesn't match the spec, precisely because it didn't author them. "Done" is earned past an independent adversary, not self-declared.

## Does coordinating models actually help?

The reviewer seat is the one I could isolate cleanly, so I ran an ablation: same loop, same coder model, **vary only the reviewer model** — a baseline versus a stronger one — on hard cases a single-shot coder had already failed.

| Reviewer model | Cases recovered |
| --- | --- |
| Baseline | 3 / 50 |
| Stronger | 6 / 50 |

A larger (~105-case) run pointed the same way: more recoveries, and **zero regressions** — not one case the baseline reviewer passed was lost by the stronger one.

I'm **not** quoting a multiplier, because when I stress-tested it the result got humbler: neal re-samples the coder each run, and re-running cases that had passed showed ~50% flip on a fresh roll. Per-case outcomes are noisy; the *size* of the gap is uncertain. What survives is the **direction** and the **asymmetry** — a stronger reviewer model helped, and never made a passing case fail. If it were only noise, you'd see losses too.

## The unflattering part

Reading ~90 cases neal couldn't solve, the large majority were **capability-ceiling**: the coder produced a complete, plausible patch the reviewer reasonably accepted, and it just didn't match the hidden tests. The loop worked; the models hit their limit. Which is the thesis restated — a single model has a ceiling, and the leverage you actually control is the coordination around it: scope the work, keep the coder focused, and let an independent model review.

## Try it

```
npm install -g @navels/neal
```

Source and docs: [github.com/navels/neal](https://github.com/navels/neal). Put whichever models you like in the planner, coder, and reviewer seats; it runs on your existing subscription auth and resumes if you kill it mid-run.

I'd genuinely value the critical read — on the coordination idea and on the benchmark methodology. Tell me where it's wrong.
