# Example loop: content refresh

Monitors a set of published pages for decay, regenerates the stale ones, and gates each
rewrite against a publish rubric before staging. For content and SEO teams.

## Goal

> Every page flagged stale has an updated draft staged for review, and every staged
> draft passes the publish rubric.

## Cadence

Weekly. A page is "stale" when it crosses a threshold you define: a traffic or ranking
drop, an outdated fact (price, stat, date), a competitor that now out-covers it, or
simply age past N months.

## Primitives used

- **Automation** — weekly scan of the tracked page set.
- **Connector** — analytics/SEO tool (MCP) to pull decay signals; CMS connector to stage.
- **Sub-agents** — `loop-maker` rewrites the page; `loop-checker` runs the rubric.
- **Skill** — point the maker at your house style / brief skill so rewrites stay on-voice.
- **State** — board tracking each page's status: fresh, flagged, drafted, staged, live.

## The prompt

```
Use the loop-engineering skill. Once a week:
1. Scan the tracked pages and flag any that crossed a staleness threshold.
2. For each flagged page, send loop-maker to produce an updated draft following our
   style skill and the current source-of-truth facts.
3. Send loop-checker to score the draft against the publish rubric (accuracy, structure,
   internal links, no fabricated stats). Only PASS drafts get staged in the CMS.
4. /goal every flagged page has a PASS draft staged or a logged reason it was skipped.
Record status per page in the state board. Stage as drafts only — never publish live.
```

## What it produces / value

A steady stream of refreshed, rubric-checked drafts waiting for a human to approve and
publish, instead of a content backlog that quietly rots. The checker prevents the most
common autonomous-content failure: confident, fabricated facts shipped at scale.

## Watch-outs

Keep the loop staging *drafts*, not publishing. Publishing is an irreversible, public
action — that stays a human decision. The rubric in the checker is doing the real work,
so make it strict and specific.
