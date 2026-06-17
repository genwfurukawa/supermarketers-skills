# Example loop: AI citation monitor (AEO / GEO)

Tracks whether your brand actually gets cited in AI answers, week over week, and queues a
content fix for every prompt where you lost a citation. This is answer engine optimization
turned into a standing instrument instead of a one-off audit.

## Goal

> This week's citation rate is logged for every tracked prompt, and every prompt where we
> lost (or never had) a citation has a content fix drafted and queued for review.

## Cadence

Weekly. You maintain a fixed set of "target prompts" — the questions your buyers actually
ask AI assistants, where you want to be the cited source.

## Primitives used

- **Automation** — weekly run over the target-prompt set.
- **Connector** — an AI answer engine via MCP (e.g. a Perplexity or search connector) to
  ask each prompt and capture which sources get cited.
- **Sub-agents** — `loop-maker` drafts the content fix for a lost-citation prompt;
  `loop-checker` confirms the fix actually answers the prompt in a citable, answer-first
  structure.
- **State** — a board logging citation rate per prompt over time, so you can see the trend
  and tie it to content changes.

## The prompt

```
Use the loop-engineering skill. Once a week:
1. For each target prompt, ask the AI answer engine and record whether our domain was
   cited, and which competitors were.
2. Log this week's citation rate per prompt to the state board (keep the history).
3. For each prompt where we lost or lack a citation, send loop-maker to draft the page or
   answer block most likely to earn it (answer-first, sourced, scannable).
4. Send loop-checker to confirm the draft directly answers the prompt and is structured
   to be quotable by an AI assistant.
5. /goal every tracked prompt has a logged citation rate AND every lost-citation prompt
   has a PASS fix queued for review.
Stage drafts only. Summarize the week's wins and losses at the end.
```

## What it produces / value

A live citation-rate dashboard plus a ranked queue of "write this to win this prompt"
fixes. This is the measurement layer most AEO work skips: it connects a specific piece of
content to a specific citation outcome, so you can prove the work and prioritize the next
piece by expected visibility gain rather than guesswork.

## Watch-outs

AI answers vary run to run, so a single miss isn't a verdict — track the rate over weeks,
not the result of one query. Don't let the loop publish; earning a citation is a content
decision a human should approve. And keep the target-prompt set honest: prompts you'd
*like* to win aren't the same as prompts your buyers actually ask.
