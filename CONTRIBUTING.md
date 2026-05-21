# Contributing

PRs and issues welcome.

## Adding a skill

A skill lives at `skills/<skill-name>/SKILL.md` with YAML frontmatter:

```markdown
---
name: skill-name
description: One sentence describing what the skill covers and when to use it.
---

# Title

Rules and examples.
```

- `<skill-name>` is kebab-case with a framework prefix (`flutter-`, `dart-`).
- One concern per skill. Split if it grows past one screen of rules.
- Examples use standard tutorial vocabulary (`Login`, `Todo`, `Counter`),
  not project-specific names.
- Code blocks are runnable Dart, not pseudocode.

Update the README table when adding a new skill.
