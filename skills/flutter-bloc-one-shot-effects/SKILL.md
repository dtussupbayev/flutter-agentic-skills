---
name: flutter-bloc-one-shot-effects
description: Pattern for one-shot UI effects (SnackBar, navigation, dialog) reacting to BLoC state transitions in flutter_bloc with freezed. Use when adding non-persistent UI reactions tied to a BLoC state change.
---

# Flutter BLoC one-shot UI effects

For SnackBar, navigation after action, toast, dialog triggered by a BLoC
state change. Based on the
[BLoC Todos tutorial](https://bloclibrary.dev/tutorials/flutter-todos/)
(`lastDeletedTodo` + `listenWhen`).

## Pattern

A nullable payload field on the data-bearing state, plus a `BlocListener`
with `listenWhen` that fires exactly when the payload changes.

```dart
@freezed
sealed class TodosState with _$TodosState {
  const factory TodosState.initial() = TodosInitial;
  const factory TodosState.loading() = TodosLoading;
  const factory TodosState.loaded({
    required List<Todo> todos,
    Todo? lastDeletedTodo,       // entity payload
    DateTime? lastRefreshedAt,   // nonce, when payload can repeat
    Exception? actionError,      // inline error that preserves data
  }) = TodosLoaded;
  const factory TodosState.error({required Exception e}) = TodosError;
}
```

```dart
// Effect-carrying emit: fresh constructor, not copyWith.
// Each emit sets only the relevant marker; other effect fields reset to null.
emit(TodosState.loaded(todos: next, lastDeletedTodo: deleted));
emit(TodosState.loaded(todos: s.todos, actionError: e));

// Data-only update (optimistic UI): copyWith is fine.
// Never set an effect field via copyWith.
emit(s.copyWith(todos: optimisticTodos));
```

```dart
MultiBlocListener(
  listeners: [
    BlocListener<TodosBloc, TodosState>(
      listenWhen: (prev, curr) {
        if (curr is! TodosLoaded || curr.lastDeletedTodo == null) return false;
        if (prev is! TodosLoaded) return true;
        return curr.lastDeletedTodo != prev.lastDeletedTodo;
      },
      listener: (context, state) {
        final s = state as TodosLoaded;
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('${s.lastDeletedTodo!.title} deleted')),
        );
      },
    ),
  ],
  child: BlocBuilder<TodosBloc, TodosState>(...),
)
```

## Rules

- UI effects only inside `BlocListener` / `BlocConsumer.listener`. Never
  in `build` or `BlocBuilder.builder`.
- One listener per effect type, grouped via `MultiBlocListener`.
- Effect = nullable payload field on the data-bearing state. Not a
  separate state variant. Not a sticky boolean.
- The payload must produce `!=` on every emit that should re-fire the
  listener. `Exception` works by identity (new instance every time);
  plain `String` does not (value equality). Add a `DateTime` nonce when
  equality is value-based.
- Effect-carrying emits go through a fresh constructor, not `copyWith`.
  `copyWith` carries old markers forward.
- Builder reads persistent data only. Never check effect fields in
  `build`. If a render branch is needed, model it as a regular field with
  its own name, not as an effect marker.
- After `await`, check `if (!context.mounted) return;`.
- BLoC / Cubit holds no `BuildContext` and never calls UI directly.

## Terminal state vs payload marker

If the current screen is gone after the effect (`Navigator.pop`,
`pushReplacement`, sheet close), a payload marker is wrong. Use a
separate terminal state variant.

```dart
@freezed
sealed class AddItemState with _$AddItemState {
  const factory AddItemState.loaded({...}) = AddItemLoaded;
  const factory AddItemState.success() = AddItemSuccess;  // terminal
  const factory AddItemState.error({required Exception e}) = AddItemError;
}

// listener: no mounted check needed when no await precedes pop
if (state is AddItemSuccess) Navigator.of(context).pop(true);
```

## Checklist

- UI effects only inside `BlocListener` / `BlocConsumer.listener`.
- Effect is a nullable payload field, not a separate state variant.
- Payload produces `!=` on every emit (verify equality semantics).
- All effect-carrying emits go through a fresh constructor.
- `listenWhen` checks both the field change and the cross-variant transition.
- Builder does not read effect fields.
- After `await`, `context.mounted` is checked.

## References

- [BLoC Todos tutorial](https://bloclibrary.dev/tutorials/flutter-todos/)
- [bloc#4073](https://github.com/felangel/bloc/issues/4073), sticky status discussion
