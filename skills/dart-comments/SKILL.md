---
name: dart-comments
description: Use when writing, changing, or reviewing documentation comments, inline comments, TODOs, or FIXMEs in Dart and Flutter source code.
---

# Dart comments

## Scope

Apply this skill to every comment in Dart source:

- documentation comments (`///`);
- implementation comments (`//`);
- `TODO` and `FIXME` comments.

It does not govern pull request comments, code review comments, commit
messages, or Markdown documentation.

## Priority

[Effective Dart: Documentation][effective-dart-docs] is authoritative. Local
repository instructions may add stricter requirements.

## Decide whether a comment is needed

Do not comment every declaration. Add a comment when it communicates
information the code cannot express clearly:

- an API contract or non-obvious behavior;
- the reason for an implementation choice;
- an invariant or hidden constraint;
- a workaround and why it remains necessary;
- a warning that prevents plausible misuse.

Documentation comments may describe what an API guarantees. Implementation
comments should primarily explain why the surrounding code is necessary. If a
comment narrates well-named code, remove it or improve the code.

## Syntax and structure

- Use `///` when documenting a declaration. The declaration does not require a
  comment when its purpose and contract are already clear.
- Use `//` for implementation context inside a function or method.
- Do not use `/* */` as documentation.
- Put a documentation comment before metadata annotations.
- Start a documentation comment with a concise one-sentence summary. Add a
  blank comment line before further detail.
- Write complete sentences with normal capitalization and punctuation.
- Reference Dart identifiers with `[brackets]` so dartdoc and IDEs link them.
- Keep comments brief. Use Markdown only when it improves documentation; do
  not use HTML or Markdown as decoration.

When comments are written in English, follow Effective Dart grammar:

- Type and library comments are noun phrases describing an instance.
- Non-boolean property comments are noun phrases.
- Boolean property comments start with `Whether`.
- Functions and methods that perform work start with a third-person verb.

For other languages, use the natural grammatical equivalent rather than
copying English sentence openings.

## Do not duplicate values from code

Do not repeat a numeric default, duration, string, threshold, or configuration
value that already exists as a literal in the source. The comment and code
will eventually diverge.

```dart
// Bad: duplicates the constructor default.
/// Waits 500 ms before confirming visibility.
final Duration readThreshold;

// Good: describes the contract without copying its current value.
/// Minimum duration of continuous visibility.
final Duration readThreshold;
```

When a comment depends on a project value, reference its identifier, such as
`[readThreshold]`, instead of copying the current literal.

## TODO and FIXME ownership

Every `TODO` and `FIXME` references a tracking link or a person:

```dart
// TODO: extract this into a helper.

// TODO(https://example.org/issues/123): extract this into a helper.
// TODO(daulet): extract this into a helper.
```

The first example is invalid because it has no owner.

## Avoid

- Decorative section dividers. Split or reorder the class instead.
- Inline change history. Put it in the commit or pull request.
- Empty documentation comments left after deleting their description.
- Comments that compensate for unclear names or unnecessary complexity.

## Review checklist

- Does the comment add a contract, reason, invariant, constraint, workaround,
  or misuse warning?
- Is `///` used for declaration documentation and `//` for implementation
  context?
- Is the first sentence a concise summary?
- Does the grammar match the documented declaration and chosen language?
- Are referenced Dart identifiers wrapped in `[brackets]`?
- Does the comment avoid duplicating literals from code?
- Does every `TODO` and `FIXME` have an owner?

## Reference

[effective-dart-docs]: https://dart.dev/effective-dart/documentation
