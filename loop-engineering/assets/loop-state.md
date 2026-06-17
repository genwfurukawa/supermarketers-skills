# LOOP_STATE.md — the spine of the loop

Copy this into the repo the loop runs on (or use a Linear board via MCP instead).
The model forgets everything between runs. This file does not. Every cycle reads it
first and writes to it last, so tomorrow's run resumes where today's stopped.

Keep it on disk, not in the prompt. Context resets; the repo persists.

---

## GOAL
<the verifiable stopping condition for this loop — checkable, not vague>

## CADENCE
<`/goal` once-until-done | `/loop <interval>` | Desktop scheduled task | cron + claude -p>

## BUDGET / EXIT
- Token ceiling: <e.g. --tokens 250K per cycle>
- Turn/time bound: <e.g. max 15 turns, or 1h>
- Fallback: <unresolved items → triage inbox / ping me>

## OPEN  (discovered, not yet done)
- [ ] <task id> — <one line> — <where it came from>

## IN PROGRESS
- [ ] <task id> — claimed by <maker> in worktree <name>

## DONE  (verified by checker)
- [x] <task id> — <what shipped> — PASS on <date>

## BLOCKED / NEEDS HUMAN
- <task id> — <why the loop can't resolve it>

## LOG  (newest first)
- <date> — <cycle summary: what was triaged, made, checked>
```

---

## Persistent scheduling (survives terminal exit)

`/loop` is session-scoped — it dies when you close Claude Code and recurring tasks expire
in ~3 days. For anything that must run indefinitely, use one of:

- **Desktop scheduled tasks** — native, persistent, no infra.
- **GitHub Actions** — run the loop on a schedule in CI; good when the work lives in a repo.
- **cron + `claude -p`** — a headless harness when you want full control. Sketch:

```bash
#!/usr/bin/env bash
# loop-cycle.sh — one cycle of a headless loop. Schedule via cron.
set -euo pipefail
cd "$REPO"

# Re-read state, do the work, write state. The prompt points at the skill + state file
# so the agent rebuilds its context from disk every run.
claude -p "Read ./LOOP_STATE.md and the loop-engineering skill. Run ONE cycle:
triage new work into OPEN, dispatch the top task to loop-maker in a fresh worktree,
have loop-checker verify it, update LOOP_STATE.md. Stop after one task or when the
GOAL condition holds. Anything unresolvable → BLOCKED with a reason." \
  --max-turns 20

# Optional: commit the state change so the next cron run sees it.
git add LOOP_STATE.md && git commit -m "loop: cycle $(date -u +%FT%TZ)" || true
```

```cron
# every weekday at 9am, local time
0 9 * * 1-5  /path/to/loop-cycle.sh >> /path/to/loop.log 2>&1
```

The script is dumb on purpose — it's just the heartbeat. The *intelligence* is the model
reading state and choosing the next action, which is exactly what separates a loop from a
plain cron job.
