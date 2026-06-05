---
name: flutter-reusable-widgets
description: How to make a reusable Flutter widget context-agnostic so it does not break the layout under some parents. Use when authoring or refactoring a shared / library widget, or when debugging layout errors like `Incorrect use of ParentDataWidget`, `RenderFlex children have non-zero flex but incoming constraints are unbounded`, `Vertical viewport was given unbounded height`, or `BoxConstraints forces an infinite width/height`.
---

# Flutter reusable widgets

A reusable widget must not silently assume its parent's layout context. Keep a presentational core context-agnostic; when a specific context is genuinely required, expose it as an explicit named adapter rather than baking it into the core. Otherwise the widget works under one parent and throws a layout error under another.

## Do not bake parent-context assumptions into the core

### ParentData widgets

`Expanded` / `Flexible` are `ParentDataWidget`s that only work inside a `Flex` (`Row` / `Column` / `Flex`); `Spacer` builds an `Expanded`, so it carries the same requirement. `Positioned` only works inside a `Stack`, `TableCell` only inside a `Table`. A `ParentDataWidget` attaches parent data that only the matching parent reads, so putting one anywhere else makes the framework throw `Incorrect use of ParentDataWidget`. It is a render contract, not a style choice. The required `Flex` / `Stack` must live inside this same widget, and the path from it to the `Expanded` / `Positioned` must contain only `StatelessWidget` / `StatefulWidget` (a `RenderObjectWidget` like `Padding` / `Container` in between breaks it). The violation is a `ParentDataWidget` whose matching parent is outside, in the consumer's tree.

```dart
// bad: works in a Column, crashes everywhere else
class PriceTag extends StatelessWidget {
  const PriceTag(this.amount, {super.key});
  final String amount;

  @override
  Widget build(BuildContext context) {
    return Expanded(child: Text(amount)); // Expanded baked into the widget
  }
}

// good: the core is just its content; the parent decides the flex role
class PriceTag extends StatelessWidget {
  const PriceTag(this.amount, {super.key});
  final String amount;

  @override
  Widget build(BuildContext context) => Text(amount);
}

// the consumer owns the layout role:
Row(children: [Expanded(child: PriceTag('\$9.99'))]);
```

### Parent type / protocol

Do not assume the parent is a `Flex`, a `Stack`, a scrollable, or a sliver context, or that it supplies a particular constraint. A widget that nests an unbounded `ListView` or a `Column` with an `Expanded` child assumes bounded constraints the consumer may not give, which throws an unbounded-constraints or overflow error.

Layout role - flex, position, scroll, safe area - is the consumer's decision. The core takes its intrinsic size, or fills the constraints the parent gives it. `double.infinity` asks for the maximum on that axis - fine when the axis is bounded, but it throws on an unbounded axis (the main axis of an unbounded `Row` / `Column`, the scroll axis of a scrollable). For example, in a vertical list `width: double.infinity` is fine but `height: double.infinity` is not.

A widget's own size and spacing at the root (`Padding`, a fixed `SizedBox`) is fine - that is its intrinsic footprint, not a parent dependency. What is banned is imposing a parent-relative role (`Expanded` / `Positioned`) or assuming a parent type.

## When context is genuinely needed: a named adapter, not an assumption

Sometimes a widget must integrate with a specific context. Expose explicit named variants and let the consumer pick:

```dart
// presentational core: a RenderBox, just the visuals - no scroll/flex assumptions
class SectionHeader extends StatelessWidget {
  const SectionHeader(this.title, {super.key});
  final String title;

  @override
  Widget build(BuildContext context) => Padding(
        padding: const EdgeInsets.all(16),
        child: Text(title, style: Theme.of(context).textTheme.titleMedium),
      );
}

// explicit adapter for a CustomScrollView - returns a sliver
class SliverSectionHeader extends StatelessWidget {
  const SliverSectionHeader(this.title, {super.key});
  final String title;

  @override
  Widget build(BuildContext context) =>
      SliverToBoxAdapter(child: SectionHeader(title));
}
```

`RenderBox` (constraints down, size up) and `RenderSliver` (scroll geometry) are different layout protocols - one widget cannot be both, so a box variant returns a box and a sliver variant returns a sliver. The vice is not context-awareness, it is a *silent* assumption: a named adapter makes the dependency visible in the API, a baked-in assumption breaks silently at the call site.

## Caveats

- **Reusable widgets only.** A public widget used by (or meant for) more than one feature - a uikit/library component, say - is reusable: the rule applies. A private (`_`-prefixed) single-use widget coupled to one parent may bake layout in - it is then not reusable. When unsure, default to reusable: over-applying costs a wrapper, under-applying causes the layout bug.
- **Refactoring an existing widget.** Removing a baked-in `Expanded` changes behavior: consumers relied on it expanding. Wrap each call site in the required layout widget (or flag it), or you break their layout.

## Checklist

- No `ParentDataWidget` (`Expanded` / `Flexible` / `Spacer` / `Positioned`) whose required `Flex` / `Stack` is outside the widget. They are fine only when the matching parent is inside the widget and directly owns them (e.g. the widget builds its own `Row` with `Expanded` children).
- No assumption that the parent is a `Flex`, `Stack`, scrollable, or sliver context.
- The core sizes intrinsically or fills given constraints; the consumer wraps it in `Expanded` / `SizedBox` / `Positioned`.
- Context integration (Scaffold / sliver / box) is an explicit named adapter over a presentational core, not baked in.
- A refactor that removes baked-in `Expanded` / `Positioned` also updates the call sites.

## References

- [ParentDataWidget](https://api.flutter.dev/flutter/widgets/ParentDataWidget-class.html) and [Expanded](https://api.flutter.dev/flutter/widgets/Expanded-class.html) - the ancestor-path contract
- [Understanding constraints](https://docs.flutter.dev/ui/layout/constraints)
- [Slivers](https://docs.flutter.dev/ui/layout/scrolling/slivers)
