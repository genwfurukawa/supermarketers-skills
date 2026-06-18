---
name: loop-engineering
description: >
  Set up autonomous agent loops in Claude Code. Define a goal with a verifiable
  stopping condition and let Claude iterate until done — on a schedule, with a
  separate verifier checking completion. Use this skill whenever the user wants to:
  create a loop, set up loop engineering, write a /goal or /loop, run a task until
  a condition passes, schedule recurring autonomous work (CI triage, log checks,
  content/doc sync, link audits), split a maker agent from a checker/verifier agent,
  convert a recurring manual prompt into a self-running loop, or stand up parallel
  sub-agents in isolated worktrees. Trigger this even when the user just says
  "make a loop for X", "run this until tests pass", "check X every morning", or
  "I want Claude to keep going without me babysitting it".
---

# Loop Engineering

Turn a recurring manual prompt-and-babysit task into a system that finds the work,
does it, checks itself, records state, and decides the next move — while you watch
instead of drive. The unit of work is no longer the prompt. It is the loop.

The whole point: **you design the loop once; you stop prompting each step.**

---

## The model in four words

- **Goal** — a recursive intent with a *verifiable* stopping condition. You state what
  "done" looks like ("all tests in tests/auth pass and lint is clean"); the agent
  iterates until that is provably true. A separate evaluator model grades each turn,
  so the agent that did the work is not the one declaring victory.
- **Loop** — the system that runs the goal on a heartbeat instead of you prompting
  turn by turn. It discovers work, dispatches it, checks the result, writes down what
  happened, and picks the next move. A loop sits one level above the *harness* (the
  environment a single agent runs inside).
- **Task** — a discrete unit the loop discovers and triages, then hands to a sub-agent,
  usually inside an isolated worktree so parallel agents do not collide on files.
- **Completion** — the verified stop. **Maker ≠ checker**: one agent writes, a second
  (different instructions, often a cheaper/faster model) verifies against the spec and
  real command output. Models grade their own homework too generously, so
  self-verification does not count as done.

The difference between a loop and a cron job: cron runs a *fixed script*; a loop runs a
*model* that reads current state and chooses its next action.

---

## THE FAST PATH — exact commands to paste

Start here. For most tasks one native command is the whole loop. Only build the full
scaffold (below) when you need parallel sub-agents, persistent scheduling, or a verifier
you can trust unattended.

### 1. Run until done (`/goal`)

Hand off an objective and walk away. A separate evaluator (Haiku by default) checks
after every turn whether the condition holds; if not, Claude keeps going.

```
/goal all tests in tests/auth pass and `npm run lint` exits clean
```

Make the condition **checkable, not vague**. "Refactor the auth module" gives the
evaluator nothing to test. "auth tests green and no eslint errors" does.

Cap the cost before you walk away:

```
/goal --tokens 250K all tests in tests/auth pass and lint is clean
```

Or bound it inside the condition: `... , max 15 turns`. Stop early with `/goal clear`.

### 2. Run on a schedule (`/loop`)

Cycle on an interval. Good for "watch this for a while" work.

```
/loop 5m check error logs and open an issue for any new stack trace
```

Session-scoped: closing the terminal cancels it, missed cycles do not stack up, and
recurring tasks auto-expire after ~3 days. A session holds up to 50 scheduled tasks.
For anything that must run tomorrow / next week / indefinitely, use **Desktop scheduled
tasks**, not `/loop`.

### 3. Scheduled AND run-until-done (combine them)

Discovery on a cadence + a hard end condition per cycle:

```
/loop every morning at 9am: triage yesterday's CI failures, then /goal each failing
suite is green or has a ticket explaining why it can't be fixed automatically
```

### 4. Parallel work (`/batch`)

Large refactor across many files → parallel agents in isolated worktrees:

```
/batch migrate every component in src/legacy/ to the new design tokens, one worktree
per directory, verify each against the existing snapshot tests before merging
```

### 5. Invoke the full scaffold (this skill)

When you want the maker/checker + state-file structure stood up for you, say:

```
Use the loop-engineering skill to set up a loop that <does X> until <condition>.
Run it <cadence>. Open a ticket / ping me for anything it can't resolve.
```

That triggers the build recipe below.

---

## Writing the goal

`/goal` only works if its stopping condition is something a machine can check. This is the
part most loops get wrong, so this skill writes every goal to one shape.

**The formula.** Every `/goal` this skill produces follows:

> `[count of items] are done AND the checker marks each PASS on [named rubric], capped at [budget]`

The count is verifiable (every prompt, every page, 5 drafts). The quality is not, so it is
delegated to the checker, which turns "good" into a PASS/FAIL the goal can read. The goal
never reads "is this good"; it reads "did the checker say PASS."

**Verifiability gate.** Before emitting a `/goal`, confirm the condition can be checked by
reading real output: a command result, a checker verdict, or a logged count. If it cannot,
the task is too vague to loop. Narrow it until the finish line is checkable. "Improve our AI
visibility" is not a goal. "Every target prompt has a logged citation result and every miss
has a checker-PASS fix" is.

**Bind the goal to the checker, never the maker.** The stopping condition references the
checker's verdict, not the maker's self-report. The agent that did the work does not get to
declare itself done.

**Always attach a budget and a bound.** Emit `--tokens` and a turn or time limit with every
goal. A goal with no exit is a bill with no ceiling.

**Routing.**
- Single verifiable objective → `/goal` alone.
- Recurring → wrap `/goal` in `/loop` (or a scheduled task) for the cadence.
- Parallel items needing isolation → the full maker/checker scaffold, with the goal reading
  checker PASS.

See `references/goal-grammar.md` for checkable-versus-vague conditions across code, content,
and citation work, plus how to write the checker rubric the goal leans on.

---

## The build recipe (full scaffold)

Follow these steps when standing up a real, unattended, multi-agent loop. Do not skip
the verifier or the state file — they are what make walking away safe.

1. **Pin the goal.** Write the stopping condition as something a script or evaluator can
   check by reading real command output. If you can't phrase a checkable condition, the
   loop isn't ready — narrow the task until you can.
2. **Define the cadence.** One-shot until done → `/goal`. Recurring while the session is
   open → `/loop`. Must survive restarts → Desktop scheduled task or push to GitHub
   Actions / a cron-driven `claude -p` harness (see `assets/loop-state.md` notes).
3. **Set up isolation.** If more than one sub-agent runs at once, give each its own
   `git worktree` (or `isolation: worktree` on the sub-agent) so edits can't collide.
4. **Create the maker.** Drop `assets/maker.md` into `.claude/agents/` and tailor it. The
   maker explores and implements one task.
5. **Create the checker.** Drop `assets/checker.md` into `.claude/agents/`. Give it
   *different* instructions and a cheaper/faster model. It verifies the maker's output
   against the spec and real test/lint/build output — never against the maker's own
   say-so.
6. **Wire the state file.** Copy `assets/loop-state.md` to the repo (e.g. `LOOP_STATE.md`
   or a board via MCP). This is the spine: it holds what's tried, what passed,
   what's still open, so tomorrow's run resumes where today's stopped. The agent forgets
   between runs; the repo doesn't.
7. **Connect the tools.** Add the MCP connectors the loop needs to *act* (issue tracker,
   chat, staging API, CMS) so it opens the PR and updates the ticket instead of just
   describing the fix.
8. **Set the exit + budget.** Always: a token ceiling (`--tokens`), a turn/time bound, and
   a fallback ("anything unresolved → triage inbox / ping me"). A loop with no exit is a
   bill with no ceiling.
9. **Watch cycle one.** Never walk away on the first run. Confirm the checker actually
   fails bad output before trusting it unattended.

---

## The six primitives (quick map)

A loop is assembled from six pieces, all native in Claude Code:

| Primitive | Job | Claude Code |
|---|---|---|
| Automations | scheduled discovery + triage | `/loop`, cron tasks, hooks, GitHub Actions, Desktop scheduled tasks |
| Worktrees | isolate parallel agents | `git worktree`, `--worktree`, `isolation: worktree` on a sub-agent |
| Skills | codify project knowledge | `SKILL.md` (so the loop doesn't re-derive your conventions every cycle) |
| Connectors | reach external tools | MCP servers + plugins |
| Sub-agents | separate maker from checker | `.claude/agents/`, agent teams, `/batch` |
| Memory | persist state between runs | state file on disk (`LOOP_STATE.md`, `CLAUDE.md`) or a board via MCP |

---

## Guardrails (read before walking away)

Three problems get *sharper* as the loop gets better, not easier:

- **Verification is still on you.** An unattended loop is an unattended mistake-maker.
  The maker/checker split is what makes "done" mean something — and even then "done" is a
  claim, not a proof. Spend a verifier sub-agent wherever a second opinion is worth paying
  for; they cost extra tokens because each runs its own model and tools.
- **Comprehension debt.** The faster the loop ships work you didn't review, the wider the
  gap between what exists and what you understand. Read what the loop produced. A smooth
  loop just grows this debt faster.
- **Cognitive surrender.** It's tempting to stop having an opinion and take whatever comes
  back. Two people run the identical loop and get opposite outcomes: one moves faster on
  work they understand, the other avoids understanding it. The loop can't tell the
  difference. You can.

Build the loop. Stay the engineer.

---

## Where this lives

Install at `~/.claude/skills/loop-engineering/` (user-level, available everywhere) or
`<repo>/.claude/skills/loop-engineering/` (project-level, shared with the team via the
repo). Move the two agent templates from `assets/` into `.claude/agents/` when you stand
up a loop.

Keep this skill general-purpose. Put the actual loop definitions — the specific goals,
cadences, and state boards for your projects — alongside your own work, and let them
reference this skill for the *how*. The conventions, build steps, and guardrails live
once, where every loop reads them. See `examples/` for ready-to-adapt loop definitions.

## Assets

- `assets/maker.md` — sub-agent definition template: explores + implements one task.
- `assets/checker.md` — verifier sub-agent template: grades output against real signals.
- `assets/loop-state.md` — state file template + headless `claude -p` harness notes.
