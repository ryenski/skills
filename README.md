# Rails Skills

Agent skills for experienced Ruby on Rails developers. Small, composable, and focused on writing better Rails code faster — not vibe coding.

> Based on [Matt Pocock's skills framework](https://github.com/mattpocock/skills).

## Quickstart

1. Run the skills.sh installer:

```bash
npx skills@latest add ryenski/skills
```

2. Run `/setup-matt-pocock-skills` in your agent. It will:
   - Ask which issue tracker you want to use (GitHub, Linear, or local files)
   - Ask what labels you apply to tickets when you triage them
   - Ask where you want to save any docs it creates

## Why These Skills Exist

These skills fix the most common failure modes when using an AI agent on a Rails codebase.

### The Agent Didn't Do What I Want

The most common failure mode is misalignment. The fix is a **grilling session** — getting the agent to ask you detailed questions before it writes a line of code.

- [`/grill-me`](./skills/productivity/grill-me/SKILL.md) — for any plan or design
- [`/grill-with-docs`](./skills/engineering/grill-with-docs/SKILL.md) — same, but also sharpens your domain language and updates ADRs

### The Code Doesn't Work

Without fast feedback loops, the agent is flying blind. The [`/tdd`](./skills/engineering/tdd/SKILL.md) skill enforces red-green-refactor with Minitest and fixtures. The [`/diagnose`](./skills/engineering/diagnose/SKILL.md) skill gives you a disciplined debugging loop with Rails-specific tooling (debugger, bullet, rack-mini-profiler, query explain).

### We Built a Ball of Mud

AI agents accelerate software entropy. [`/rails-patterns`](./skills/engineering/rails-patterns/SKILL.md) gives you a decision tree for where logic belongs — model, service object, form object, decorator, or strategy. [`/improve-codebase-architecture`](./skills/engineering/improve-codebase-architecture/SKILL.md) helps rescue a codebase that's already gone wrong.

## Reference

### Engineering

Skills I use daily for code work.

- **[diagnose](./skills/engineering/diagnose/SKILL.md)** — Disciplined diagnosis loop for hard bugs and performance regressions, extended with Rails-specific tooling (debugger, bullet, rack-mini-profiler, query explain).
- **[grill-with-docs](./skills/engineering/grill-with-docs/SKILL.md)** — Grilling session that challenges your plan against the existing domain model, sharpens terminology, and updates `CONTEXT.md` and ADRs inline.
- **[improve-codebase-architecture](./skills/engineering/improve-codebase-architecture/SKILL.md)** — Find deepening opportunities in a codebase, informed by the domain language in `CONTEXT.md` and the decisions in `docs/adr/`.
- **[rails-patterns](./skills/engineering/rails-patterns/SKILL.md)** — Object-oriented design patterns for Rails: service objects, form objects, decorators, strategies, observers, and when to extract from models.
- **[setup-matt-pocock-skills](./skills/engineering/setup-matt-pocock-skills/SKILL.md)** — Scaffold the per-repo config (issue tracker, triage label vocabulary, domain doc layout) that the other engineering skills consume.
- **[tdd](./skills/engineering/tdd/SKILL.md)** — Test-driven development with Minitest and fixtures. Red-green-refactor, one vertical slice at a time.
- **[to-issues](./skills/engineering/to-issues/SKILL.md)** — Break any plan, spec, or PRD into independently-grabbable issues using vertical slices.
- **[to-prd](./skills/engineering/to-prd/SKILL.md)** — Turn the current conversation context into a PRD and submit it as an issue.
- **[triage](./skills/engineering/triage/SKILL.md)** — Triage issues through a state machine of triage roles.
- **[zoom-out](./skills/engineering/zoom-out/SKILL.md)** — Tell the agent to zoom out and give broader context on an unfamiliar section of code.
- **[prototype](./skills/engineering/prototype/SKILL.md)** — Build a throwaway prototype to flush out a design.

### Productivity

General workflow tools, not code-specific.

- **[caveman](./skills/productivity/caveman/SKILL.md)** — Ultra-compressed communication mode. Cuts token usage ~75% by dropping filler while keeping full technical accuracy.
- **[grill-me](./skills/productivity/grill-me/SKILL.md)** — Get relentlessly interviewed about a plan or design until every branch of the decision tree is resolved.
- **[write-a-skill](./skills/productivity/write-a-skill/SKILL.md)** — Create new skills with proper structure, progressive disclosure, and bundled resources.

### Misc

Tools I keep around but rarely use.

- **[git-guardrails-claude-code](./skills/misc/git-guardrails-claude-code/SKILL.md)** — Set up Claude Code hooks to block dangerous git commands (push, reset --hard, clean, etc.) before they execute.
