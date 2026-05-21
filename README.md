# flutter-agentic-skills

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Agent skills for Flutter and Dart projects. Drop them into your AI
coding assistant so it follows the same conventions you do.

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

## Contributing

PRs and issues welcome. See [`CONTRIBUTING.md`](CONTRIBUTING.md).

## License

[MIT](LICENSE).
