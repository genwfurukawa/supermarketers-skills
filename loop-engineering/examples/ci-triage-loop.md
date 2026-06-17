# Example loop: CI failure triage

The canonical loop. Every weekday morning it finds yesterday's CI failures, drafts a
fix for each in its own worktree, and verifies before anything merges.

## Goal

> Every failing CI suite from the last run is green, or has a ticket explaining why it
> can't be fixed automatically.

## Cadence

Weekday mornings. Session-scoped `/loop` if you keep Claude Code open; a Desktop
scheduled task or GitHub Actions if it must run while you're away.

## Primitives used

- **Automation** — scheduled discovery of new CI failures.
- **Worktree** — one isolated checkout per fix so parallel drafts don't collide.
- **Sub-agents** — `loop-maker` drafts the fix; `loop-checker` re-runs the suite.
- **Connector** — issue tracker (MCP) to open tickets the loop can't resolve.
- **State** — `LOOP_STATE.md` tracks which failures are fixed, ticketed, or open.

## The prompt

```
Use the loop-engineering skill. Every weekday at 9am:
1. Pull the failing suites from the most recent CI run.
2. For each failure, open an isolated worktree and send loop-maker to draft a fix.
3. Send loop-checker to re-run that suite and verify it passes without breaking others.
4. /goal every failing suite is green OR has a ticket explaining why it can't be
   auto-fixed. Cap at --tokens 200K per cycle.
Record everything in LOOP_STATE.md. Anything still red after the budget → ticket + ping me.
```

## What it produces / value

Green builds by the time you open your laptop, plus a paper trail of what was fixed and
what genuinely needs a human. Removes the slow, context-heavy work of triaging other
people's failures by hand.

## Watch-outs

Run cycle one yourself. Confirm `loop-checker` actually fails a bad patch before you trust
it to merge unattended. Flaky tests will burn budget retrying themselves — quarantine
known-flaky suites out of the loop's scope.
