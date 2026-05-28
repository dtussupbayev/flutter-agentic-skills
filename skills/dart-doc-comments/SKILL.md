---
name: dart-doc-comments
description: Doc-comment and inline comment style for Dart and Flutter. Use when writing or reviewing comments, doc-strings on classes, methods, fields, or top-level functions.
---

# Dart doc-comments

## Priority

[Effective Dart: Documentation][effective-dart-docs] is the authoritative
source for all comment and doc-string rules. If anything in this skill ever
conflicts with Effective Dart, **Effective Dart wins**.

This skill lists:

1. Which Effective Dart rules to respect (so an agent has the surface in
   context without fetching the canonical page each time).
2. Deltas - the few places this style goes stricter than Effective Dart,
   each with a documented reason.

## Effective Dart rules in force

The full Effective Dart documentation guide applies. At minimum, respect:

- `///` on declarations (classes, fields, methods, top-level functions);
  `//` inline. Never `/* */` for documentation.
- Format comments like sentences: capitalize first word, end with `.`,
  `!`, or `?`.
- Start a doc comment with a single-sentence summary, then a blank line
  before further detail.
- Phrasing matches the kind of the declaration:
  - Non-boolean property or variable: **noun phrase**.
  - Boolean property or variable: **`Whether ...`**.
  - Function or method that does work: **third-person verb**.
  - Type or library: **noun phrase describing an instance**.
- AVOID redundancy with surrounding context - focus on what the reader
  doesn't already know from the signature, type, or enclosing class.
- Reference identifiers with `[brackets]` so dartdoc and IDEs link them.
- Be brief. Use markdown for code blocks (backtick fences with language
  tag), not for decoration. No HTML.
- Doc comment goes **before** metadata annotations.

For full text, examples, and edge cases - see the link above.

## Deltas

### DO NOT embed values that live in code

Effective Dart's "AVOID redundancy" generalises to this. Called out
separately because it is the single most common way comments rot.

If a comment mentions a numeric default, a string, or any value that also
exists as a literal in the source, the value will diverge when the literal
changes and the comment doesn't.

Bad:

```dart
// Default is 0.3 - fraction of viewport before snap commits.
final double commitFraction;
```

(The actual default is `0.2` two lines below - the comment already lies.)

Good:

```dart
/// Fraction of the viewport that the user must drag past the page boundary
/// before a release without fling-velocity snaps to the next page.
final double commitFraction;
```

If a comparison is needed, compare to a **stable framework constant**
(`PageScrollPhysics` default is `0.5`) - that value does not change with
refactors. Never compare to a current value of your own.

### DO NOT use em-dashes

Use `-`, `:`, or `,`. Em-dash (`—`) is visually indistinguishable from a
hyphen in many terminal fonts and complicates grep / git diff review.

### DO attach an owner to every TODO

Every `TODO` references a tracking link or a person name. Anonymous TODOs
become permanent regrets.

```dart
// Bad.
// TODO: extract this into a helper.

// Good (link).
// TODO(https://example.org/issues/123): extract this into a helper.

// Good (name).
// TODO(daulet): extract this into a helper.
```

## Banned outright

- Decorative section dividers: `// === Foo ===`, `// --- Bar ---`,
  `// ───── Foo ─────`. If a class needs section labels, split the class.
- Inline change history: `// Was 200, increased to 240 in PR #123.` This
  belongs in the PR body or commit message.
- Empty `///` doc comments left from a removed description.

## Quick checklist

- `///` on declarations, `//` inline?
- Single-sentence summary on first line, blank line after?
- Phrasing matches kind (noun for property, "Whether" for bool, third-person
  verb for function)?
- Free of redundancy - nothing the signature already says?
- Referenced symbols in `[brackets]`?
- No code-literal duplication?
- No em-dashes?
- Every TODO has an owner?

## References

- [Effective Dart: Documentation](https://dart.dev/effective-dart/documentation)

[effective-dart-docs]: https://dart.dev/effective-dart/documentation
