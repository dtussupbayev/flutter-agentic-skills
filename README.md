# flutter-agentic-skills

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![skills.sh](https://img.shields.io/badge/skills.sh-listed-blue.svg)](https://skills.sh/dtussupbayev/flutter-agentic-skills)

Skills for Flutter and Dart projects, formatted for AI coding assistants
that read `.agents/skills` (Claude Code, Codex CLI, Cursor, OpenCode,
others).

Inspired by [`flutter/skills`](https://github.com/flutter/skills),
[`dart-lang/skills`](https://github.com/dart-lang/skills),
[`MADTeacher/mad-agents-skills`](https://github.com/MADTeacher/mad-agents-skills).

## Skills

| Skill | Topic |
| ----- | ----- |
| [`flutter-bloc-naming`](skills/flutter-bloc-naming/SKILL.md) | Event, state, and handler naming in `flutter_bloc` + `freezed`. |
| [`flutter-bloc-one-shot-effects`](skills/flutter-bloc-one-shot-effects/SKILL.md) | SnackBar, navigation, dialog reacting to BLoC state transitions. |
| [`dart-doc-comments`](skills/dart-doc-comments/SKILL.md) | Doc-comment and inline comment style. |

## Install

```bash
npx skills add dtussupbayev/flutter-agentic-skills --skill '*' --agent universal
```

The `--agent universal` flag puts the skills in the standard `.agents/skills`
folder that Claude Code, Codex CLI, Cursor, OpenCode, and most other agents
read from. Add `-g` to install globally instead of into the current
directory.

## Update

```bash
npx skills update
```

Updates all installed skills to the latest versions. Pass a skill name to
update one (`npx skills update dart-doc-comments`).
