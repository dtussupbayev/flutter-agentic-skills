---
name: flutter-bloc-naming
description: Naming conventions for events, states, and handlers in flutter_bloc with freezed. Use when adding or renaming BLoC events, state variants, or event handlers.
---

# Flutter BLoC naming

For `flutter_bloc` + freezed. Extends
[the official BLoC naming convention](https://bloclibrary.dev/naming-conventions/)
with stricter event names and a freezed-aware factory shape.

## Events

### Public class names

Freezed generates `= _Load` for a union member by default. Override it
to a public class so the UI can call the constructor directly without
going through the `Event.factory()` indirection.

```dart
@freezed
sealed class LoginEvent with _$LoginEvent {
  const factory LoginEvent.started() = LoginStarted;
  const factory LoginEvent.usernameChanged(String value) = LoginUsernameChanged;
  const factory LoginEvent.passwordChanged(String value) = LoginPasswordChanged;
  const factory LoginEvent.submitted() = LoginSubmitted;
}
```

```dart
bloc.add(LoginStarted());                       // good
bloc.add(LoginEvent.started());                 // avoid (extra indirection)
```

### Past tense, domain-neutral

An event describes what happened in the domain, not what the UI did. A
domain event survives the button being swapped for a swipe, a back-press,
or a programmatic trigger.

```dart
// good
LoginSubmitted, LogoutRequested, SessionFinished
UsernameChanged, ProfileTabSelected, FavoriteToggled
TimerTicked, SyncTicked
Started, Initialized, ItemsRefreshed

// bad: imperative, reads like a command
LoadData, StartSession, CancelSession

// bad: pinned to a UI trigger
SubmitButtonPressed, LaunchButtonClicked, CancelTapped
```

Common shapes:

- User action: `<Verb>Requested` (trigger-neutral) or `<Thing>Submitted`
  for a form submit.
- Value change: `<Field>Changed`, `<Thing>Selected`, `<Thing>Toggled`.
- System: `<Thing>Ticked`, `<Thing>Started`, `<Thing>Finished`.
- Screen lifecycle: `Started` (the BLoC community default for "ready,
  load data") and `Closed`.

`Pressed`, `Clicked`, `Tapped` are forbidden. This is stricter than the
official convention, which allows `Pressed`. The trade-off is fewer
renames when the UI changes.

### Feature prefix

To avoid collisions across BLoCs and to keep events globally readable:

```dart
LoginStarted              // good
ProfileEditorStarted      // good
Started                   // too generic
```

### Factory name matches class name

```dart
const factory LoginEvent.started() = LoginStarted;
const factory LoginEvent.usernameChanged(String value) = LoginUsernameChanged;
```

`started` ↔ `Started`, `usernameChanged` ↔ `UsernameChanged`. No surprises.

### Sealed

Events are `sealed` (requires freezed 3.0+) so exhaustive switches catch
unhandled cases at compile time.

```dart
@freezed
sealed class LoginEvent with _$LoginEvent { ... }
```

## States

Sealed class, public variants, present-tense names. The name describes
the current state, not the transition into it.

```dart
@freezed
sealed class LoginState with _$LoginState {
  const factory LoginState.initial() = LoginInitial;
  const factory LoginState.loading() = LoginLoading;
  const factory LoginState.success() = LoginSuccess;
  const factory LoginState.failure({required Exception e}) = LoginFailure;
}
```

```dart
// good: present-tense, current state
Initial, Loading, Loaded, Saving, Success, Failure, Empty
Setup, Running, Paused, Finished, Cancelled

// bad: reads like an event
LoadSucceeded, StartedLoading, ShowError
```

## Handlers

Handler name is `_on<EventSuffix>`, where `EventSuffix` is the part of
the event class after the feature prefix. No feature prefix on the
handler itself, it is already local to the BLoC.

```dart
on<LoginStarted>(_onStarted);
on<LoginUsernameChanged>(_onUsernameChanged);
on<LoginSubmitted>(_onSubmitted);

Future<void> _onStarted(LoginStarted event, Emitter<LoginState> emit) async { ... }
void _onUsernameChanged(LoginUsernameChanged event, Emitter<LoginState> emit) { ... }
```

## Checklist

- Event class is public, not `_Load`.
- UI calls the constructor directly, not the factory.
- Event is in past tense.
- Event has no `Pressed` / `Clicked` / `Tapped` suffix.
- Event has a feature prefix (`LoginStarted`, not bare `Started`).
- Event class is `sealed`.
- State class is `sealed`, state variants are public.
- Handler is `_on<EventSuffix>`, no feature prefix.

## References

- [Official BLoC naming convention](https://bloclibrary.dev/naming-conventions/)
