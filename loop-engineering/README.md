# loop-engineering

A [Claude Code](https://www.claude.com/product/claude-code) skill for **loop engineering**: stop prompting an agent step by step, and instead design a loop that prompts the agent for you. You define a goal with a verifiable stopping condition; Claude iterates until it's met, on a schedule, with a separate agent verifying completion.

> The leverage point moved. Your job is to write loops, not to operate the agent.

Loop engineering is the pattern named by Addy Osmani and championed by the people who build these tools. This skill packages it into something you drop into Claude Code and reuse.

## What it gives you

- A four-word mental model: **goal → loop → task → completion**.
- Copy-paste commands for the native `/goal`, `/loop`, and `/batch` primitives.
- A full **maker/checker** scaffold (`assets/`) so an unattended loop verifies its own output with a *different* agent — the single design choice that makes walking away safe.
- A **state-file** template plus a headless `claude -p` + cron harness for loops that must survive a terminal exit.
- Worked **examples** for dev and marketing workflows.

## Install

User-level (available in every project):

```bash
git clone https://github.com/<you>/loop-engineering.git \
  ~/.claude/skills/loop-engineering
```

Or project-level (shared with your team via the repo):

```bash
git clone https://github.com/<you>/loop-engineering.git \
  <your-repo>/.claude/skills/loop-engineering
```

When you stand up a loop, copy the two agent templates into your agents directory:

```bash
cp ~/.claude/skills/loop-engineering/assets/maker.md   .claude/agents/loop-maker.md
cp ~/.claude/skills/loop-engineering/assets/checker.md .claude/agents/loop-checker.md
```

Requires a Claude Code build with `/goal` (shipped v2.1.139, May 2026).

## Quick start

Run until a checkable condition holds, with a cost ceiling:

```
/goal --tokens 250K all tests in tests/auth pass and `npm run lint` exits clean
```

Cycle on a schedule (session-scoped):

```
/loop 5m check error logs and open an issue for any new stack trace
```

Stand up the full maker/checker scaffold by asking for it in plain language:

```
Use the loop-engineering skill to set up a loop that <does X> until <condition>.
Run it <cadence>. Open a ticket for anything it can't resolve.
```

## The model in four words

| Term | Meaning |
|---|---|
| **Goal** | A recursive intent with a *verifiable* stopping condition. A separate evaluator grades each turn, so the agent that did the work isn't the one declaring victory. |
| **Loop** | The system that runs the goal on a heartbeat instead of you prompting turn by turn: discover, dispatch, check, record, repeat. |
| **Task** | A discrete unit the loop discovers and hands to a sub-agent, usually in an isolated worktree so parallel agents don't collide. |
| **Completion** | The verified stop. **Maker ≠ checker**: one agent writes, a different one verifies against real command output. |

The difference between a loop and a cron job: cron runs a fixed script; a loop runs a model that reads current state and chooses its next action.

## Examples

The [`examples/`](./examples) directory has ready-to-adapt loop definitions:

- [`ci-triage-loop.md`](./examples/ci-triage-loop.md) — every weekday morning, triage the last run's CI failures, draft a fix per failure in an isolated worktree, and verify before merging.
- [`content-refresh-loop.md`](./examples/content-refresh-loop.md) — monitor a content set for decay, regenerate stale pages, gate each against a publish rubric before staging.
- [`citation-monitor-loop.md`](./examples/citation-monitor-loop.md) — track whether your brand gets cited in AI answers (answer engine optimization), and queue a content fix for every prompt where you lost a citation.

## Guardrails

Three problems get *sharper* as the loop gets better, not easier:

1. **Verification is still on you.** An unattended loop is an unattended mistake-maker. The maker/checker split makes "done" mean something; it doesn't make it a proof.
2. **Comprehension debt.** The faster the loop ships work you didn't review, the wider the gap between what exists and what you understand.
3. **Cognitive surrender.** Two people run the identical loop and get opposite outcomes: one moves faster on work they understand, the other avoids understanding it. The loop can't tell the difference. You can.

**Build the loop. Stay the engineer.**

## Credits

The loop-engineering pattern and the maker/checker framing draw on public writing by Addy Osmani, Peter Steinberger, and the Claude Code team, plus Anthropic's research on long-running agents. This repo packages those ideas as a reusable skill; it is not affiliated with or endorsed by Anthropic.

## License

[MIT](./LICENSE)
