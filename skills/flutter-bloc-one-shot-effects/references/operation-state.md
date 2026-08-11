# Ignoring late async completions

Use this pattern when a request can finish after a timeout, retry, or external
state transition. If the handler cannot be replaced while it waits, remove
`attemptId` and `owns`.

```dart
import 'package:bloc/bloc.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:freezed_annotation/freezed_annotation.dart';

part 'operation_state.freezed.dart';

@freezed
sealed class LoginOperation with _$LoginOperation {
  const LoginOperation._();

  const factory LoginOperation.idle() = LoginIdle;
  const factory LoginOperation.running({required int attemptId}) = LoginRunning;
  const factory LoginOperation.success() = LoginSuccess;
  const factory LoginOperation.failure({required Exception error}) =
      LoginFailure;

  bool get isRunning => this is LoginRunning;

  bool owns(int attemptId) => switch (this) {
    LoginRunning(attemptId: final current) => current == attemptId,
    _ => false,
  };
}

@freezed
abstract class LoginState with _$LoginState {
  const factory LoginState({
    @Default(LoginOperation.idle()) LoginOperation operation,
  }) = _LoginState;
}

sealed class LoginEvent {
  const LoginEvent();
}

final class LoginSubmitted extends LoginEvent {
  const LoginSubmitted();
}

final class LoginCancelled extends LoginEvent {
  const LoginCancelled();
}

abstract interface class AuthRepository {
  Future<void> login();
}

final class LoginBloc extends Bloc<LoginEvent, LoginState> {
  LoginBloc(this._repository) : super(const LoginState()) {
    on<LoginSubmitted>(_onSubmitted);
    on<LoginCancelled>(_onCancelled);
  }

  final AuthRepository _repository;
  int _attemptSequence = 0;

  void _onCancelled(LoginCancelled event, Emitter<LoginState> emit) {
    if (!state.operation.isRunning) return;
    emit(state.copyWith(operation: const LoginOperation.idle()));
  }

  Future<void> _onSubmitted(
    LoginSubmitted event,
    Emitter<LoginState> emit,
  ) async {
    if (state.operation.isRunning) return;

    final attemptId = ++_attemptSequence;
    emit(
      state.copyWith(operation: LoginOperation.running(attemptId: attemptId)),
    );

    try {
      await _repository.login();
      final latest = state;
      if (emit.isDone || !latest.operation.owns(attemptId)) return;
      emit(latest.copyWith(operation: const LoginOperation.success()));
    } on Exception catch (error) {
      final latest = state;
      if (emit.isDone || !latest.operation.owns(attemptId)) return;
      emit(latest.copyWith(operation: LoginOperation.failure(error: error)));
    }
  }
}

final class LoginActions extends StatelessWidget {
  const LoginActions({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocListener<LoginBloc, LoginState>(
      listenWhen: (previous, current) {
        return switch (current.operation) {
          LoginSuccess() => previous.operation is! LoginSuccess,
          LoginFailure() => previous.operation is! LoginFailure,
          _ => false,
        };
      },
      listener: (context, state) {
        switch (state.operation) {
          case LoginSuccess():
            Navigator.of(context).pushReplacementNamed('/home');
          case LoginFailure(:final error):
            ScaffoldMessenger.of(
              context,
            ).showSnackBar(SnackBar(content: Text(error.toString())));
          default:
            break;
        }
      },
      child: BlocSelector<LoginBloc, LoginState, bool>(
        selector: (state) => state.operation.isRunning,
        builder: (context, isRunning) {
          return Row(
            children: [
              ElevatedButton(
                onPressed: isRunning
                    ? null
                    : () =>
                          context.read<LoginBloc>().add(const LoginSubmitted()),
                child: const Text('Sign in'),
              ),
              TextButton(
                onPressed: isRunning
                    ? () =>
                          context.read<LoginBloc>().add(const LoginCancelled())
                    : null,
                child: const Text('Cancel'),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

The state guard rejects duplicate taps. Cancellation releases the UI without
cancelling the request already on the wire. If the user starts another
attempt, the old completion is still alive but no longer owns the operation.
