---
name: dart-doc-comments
description: Doc-comment and inline comment style for Dart. Use when writing or reviewing comments, doc-strings on classes, methods, fields, or top-level functions.
---

# Dart doc-comments

Rules for comments that do not rot when the code changes. Each rule
exists to prevent the same class of failure: the comment and the code
drift apart and the comment becomes a lie.

## WHY, never WHAT

Do not restate what a well-named function or type already shows.
Class- and method-level descriptions of behaviour are banned. Keep
one-line WHYs for non-obvious invariants, hidden constraints, or
workarounds.

## Do not duplicate values that live in code

If a comment mentions a numeric default, a string, or any other value
that also exists as a literal in the source, that value will eventually
diverge.

```dart
// bad: comment lies the moment the literal changes
// Default is 0.3, fraction of viewport before snap commits.
final double commitFraction;

// good: literal lives in the constructor, comment carries semantics
/// Fraction of the viewport the user must drag past the page boundary
/// before a release without fling-velocity snaps to the next page.
final double commitFraction;
```

If you need a reference point, compare to a framework constant, not to
your current value:

```dart
/// Fraction of the viewport before snap commits.
/// Standard [PageScrollPhysics] uses 0.5.
final double commitFraction;
```

A framework constant will not change with your refactors; your current
value will.

## Use `///` on declarations, `//` inline

`///` puts the comment into dartdoc, IDE hover, and tooling. `//` is for
inline WHY notes inside a function body.

```dart
// bad
// How many seconds before auto-finish kicks in.
static const int autoFinishSec = 5;

// good
/// How many seconds before auto-finish kicks in.
static const int autoFinishSec = 5;
```

## `[Symbol]` references in dartdoc

When referencing a class, function, or field by name inside a doc-comment,
wrap it in square brackets. Dartdoc renders it as a link; the IDE makes
it click-through.

```dart
/// Like [PageScrollPhysics] but with a lower distance threshold.
```

## A doc-string is not a changelog

"Changed from 0.5 to 0.3 because users complained about swipe sensitivity"
goes in the PR body or commit message, not in the source. If the code
changes again, the comment becomes a museum exhibit.

## Banned

- Decorative section dividers (`// === Foo ===`, `// --- Bar ---`). If a
  class needs section labels, split the class.
- Restating the signature: `/// Returns the user's id.` on a method
  named `getUserId()`.
- Inline change history (`// Was 200, increased to 240 in PR #123`).
- Empty `///` doc-comments left from a removed description.
- `TODO` without an owner or link.

## Checklist

- Does the comment say WHY, not WHAT?
- Does it duplicate a value from the code? If yes, remove the value.
- Is it `///` on declarations, `//` inline?
- Are referenced symbols wrapped in `[brackets]`?
- Is the prose language consistent with the rest of the codebase?
