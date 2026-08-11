---
name: flutter-bloc-one-shot-effects
description: "Use when a flutter_bloc flow drives a SnackBar, dialog, navigation, or haptics and the same outcome can repeat, a BlocListener can remount, or async work can finish out of order."
---

# Flutter BLoC one-shot effects

First decide how long a reaction must remain available. Then use the simplest
state that preserves that behavior. Keep operation outcomes in bloc state and
let the UI turn state transitions into SnackBars, dialogs, navigation, and
haptics.

## Choose when the reaction may be dropped

| Situation | Model |
| --- | --- |
| A tap changes only local presentation | Widget callback or local widget state |
| A value affects rendering, such as progress or validation | Regular bloc state read by the builder |
| An operation outcome matters only on the current route | Typed operation state and `BlocListener` |
| A domain fact the UI consumes and may undo, such as a last deleted todo | Nullable payload field, cleared by the consuming event |
| A listener may remount while its bloc survives | Pending outcome with an ID, cleared after the UI handles it |
| An outcome must outlive the bloc | Persist the outcome outside the bloc |
| Navigation follows durable auth or onboarding state | Declarative router |
| Analytics or a repository write | Domain or data-layer work, not a UI effect |

`BlocListener` observes changes after it subscribes; it does not receive the
current state on mount. A route-local listener sees the reaction only while it
is mounted. Leaving the route drops it by design.

## Route-local default

Keep one typed operation field in the screen state:

1. Start an attempt by entering `running`.
2. Perform the work.
3. Re-read the current bloc state and enter `success` or `failure`.
4. In the UI, listen for entry into a terminal variant.

If success closes or replaces the screen and no data must remain renderable,
use a terminal whole-state variant instead of an operation field.

The `running` transition separates repeated equal failures, so no timestamp
or random nonce is needed. Clear a validation value before validating a new
attempt when the same validation result must be observable again.

Use one shared operation field only when foreground operations must block one
another. Every foreground handler must then reject work while the field is
running. Operations that may overlap need separate fields. Background writes
must not clear or overwrite a foreground operation.

## Track attempts only when older work can finish late

Do not add attempt ids to synchronous validation or work that cannot be
replaced while running. Add a monotonic attempt id only when a timeout, retry,
sync update, or another event can replace an operation before its `await`
completes.

- Check `emit.isDone` after `await`.
- Read the latest state rather than emitting from the starting snapshot.
- Apply the completion only while the latest operation owns the attempt id.
- If sync moves the screen to another state variant, decide whether it
  completes the current operation or carries that operation into the new
  state.
- Bloc handlers are concurrent by default. Use `droppable`, `restartable`, or
  `sequential` only when that behavior fits the operation.
- For ordered writes with immediate UI, update state synchronously and enqueue
  an internal persist event with an immutable request snapshot under
  `sequential()`.
- `sequential()` orders one bloc instance. Several clients need a server-side
  revision check, compare-and-set, idempotency key, or queue.

See [the complete example](references/operation-state.md) when older work can
finish after the state has moved on.

## Rules

- Put bloc-driven imperative reactions in `BlocListener` or
  `BlocConsumer.listener`, never in `build` or `BlocBuilder.builder`.
- Keep `BuildContext`, navigation, dialogs, and SnackBars out of the bloc.
- A separate non-replay effect stream has the same late-listener gap. If it
  replays, it also needs a pending value and a way to clear it, so it is no
  simpler than keeping that lifecycle in state.
- Replace the whole typed operation field with `copyWith`; do not ban
  `copyWith` for unrelated data updates.
- When the UI may remount, show an existing pending outcome on mount, listen
  for later values, then clear the matching ID. Showing and clearing are not
  atomic, so tolerate duplicates.

## Verify behavior

- The same validation or failure can be presented on consecutive attempts.
- A stale completion and a stale timeout cannot overwrite a newer attempt.
- Overlapping operations do not clear each other's progress or guards.
- A listener mounted after the transition either misses or receives the
  reaction as intended.
- Ordered writes leave the server with the latest local choice.
- The bloc exposes facts and operation outcomes, not UI commands.

## References

- [BlocListener behavior](https://bloclibrary.dev/flutter-bloc-concepts/)
- [bloc_concurrency transformers](https://pub.dev/packages/bloc_concurrency)
- [Flutter Command pattern](https://docs.flutter.dev/app-architecture/design-patterns/command)
