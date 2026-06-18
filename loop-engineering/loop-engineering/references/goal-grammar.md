# Goal grammar: writing verifiable stopping conditions

A `/goal` is only as good as its stopping condition. The condition must be checkable by
reading real output. This file collects patterns to copy.

## The shape

```
[count of items] are done AND the checker marks each PASS on [named rubric], capped at [budget]
```

Countable plus delegated quality. The count proves the work happened; the checker proves it
is good. The goal never judges quality itself.

## Checkable vs vague

| Vague (do not use) | Checkable (use) |
|---|---|
| Improve our AI visibility | Every target prompt has a logged citation result, and every prompt where we are not cited has a fix the checker marks PASS on the answer-format rubric |
| Make the blog better | Every page flagged stale has an updated draft that passes the publish rubric and breaks no internal links |
| Write some LinkedIn posts | The transcript has produced 5 drafts, each PASS on the voice rubric |
| Fix the bugs | All tests in tests/auth pass and `npm run lint` exits clean |
| Clean up the docs | Every markdown file in /docs has zero broken links, confirmed by the link checker |
| Answer common questions | Every harvested question has an answer block under 60 words that leads with the direct answer and is fact-checked against the source doc |

The tell: a vague condition describes a vibe; a checkable one names a count and a test.

## Writing the checker rubric

Because the goal leans on "checker says PASS," the rubric is where quality actually lives.
Keep every rubric item binary, answerable yes/no by reading the output:

- Leads with the direct answer in the first sentence: yes/no
- Under 60 words: yes/no
- Every claim sourced: yes/no
- No em dashes, no AI filler: yes/no
- Internal links resolve: yes/no

If a rubric item cannot be answered yes/no from the output, rewrite it until it can.

## Budgets

Always cap. Pick one or more:

- `--tokens 150K` per run
- a turn limit written into the condition: `... , max 15 turns`
- a time bound for scheduled loops

## Stop-behavior tuning

- Stops too early with half-done work: the condition is too loose. Add the missing count or
  rubric item.
- Never stops, burns budget: the condition is unverifiable or too strict. Make the check
  readable, or relax the bar.
