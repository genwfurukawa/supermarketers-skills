# supermarketers-skills

Open-source [Claude Code](https://www.claude.com/product/claude-code) skills from [SuperMarketers](https://supermarketers.ai) — practical, reusable automation for AI visibility, AEO/GEO, and the agent infrastructure behind it.

Each skill is a self-contained folder with its own `SKILL.md`, assets, and examples. Use one, use several, or fork the lot.

## Skills

| Skill | What it does |
|---|---|
| [`loop-engineering`](./loop-engineering) | Set up autonomous agent loops: define a goal with a verifiable stopping condition and let Claude iterate until done, on a schedule, with a separate verifier checking completion. Includes maker/checker templates, a state-file pattern, and worked examples for CI triage, content refresh, and AI-citation monitoring. |

More on the way.

## Install a skill

Skills live in `~/.claude/skills/` (available everywhere) or `<your-repo>/.claude/skills/` (shared with a team via the repo). Clone the collection and copy the skill you want:

```bash
git clone https://github.com/genwfurukawa/supermarketers-skills.git
cp -r supermarketers-skills/loop-engineering ~/.claude/skills/loop-engineering
```

Then open Claude Code in any project and the skill is available. Each skill's own README covers its specifics.

## Repo layout

```
supermarketers-skills/
├── README.md            <- you are here (collection index)
├── LICENSE              <- MIT, covers every skill
└── <skill-name>/        <- one folder per skill
    ├── SKILL.md         <- the skill Claude loads
    ├── assets/          <- templates instantiated into output
    ├── examples/        <- worked, ready-to-adapt patterns
    └── evals/           <- test prompts for the skill
```

## License

[MIT](./LICENSE). Built by Gen Furukawa. Not affiliated with or endorsed by Anthropic.
