### Introduction
This section discusses a few guidelines and things to watch out for when using Kotlin coroutines. The list is by no means exhaustive, and shall be progressively updated as the need arises.

We rely very heavily on [Kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines) library which provides a very rich set of APIs that cover almost all of our use cases. This library is built around [structured concurrency](https://en.wikipedia.org/wiki/Structured_concurrency)  which helps us write safe and good quality code.

We also provide a [pandora-coroutines](https://github.com/deliveryhero/pd-metalcore-android/tree/master/coroutines) artifact from metalcore that contains a few utilities to help ease the use of coroutines in our codebase. In addition, the official [Android documentation](https://developer.android.com/kotlin/coroutines) provides very good resources regarding coroutines in Android development.

The content of this text is specifically adapted for our codebase, and we should ensure to follow them accordingly. However, for anything not mentioned herein, feel free to fall back to [Android best practices](https://developer.android.com/kotlin/coroutines/coroutines-best-practices), and ultimately use the official [Kotlinx.coroutines guidelines](https://kotlinlang.org/docs/coroutines-guide.html) when all else fails

### General notes
Coroutines are _safe_ and can be used in all layers of development, including the UI layer.

OkHttpClient and Retrofit instances exposed from the app module have been properly configured to support coroutines, especially for [unified API error handling](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/unified-error-handling/). Whenever there's a need to use customized version of these components, make sure to extend from the standard ones. The instances may be retrieved via DI, using `@PandoraApi` and `@DiscoApi` qualifiers accordingly

Always ensure to use the right scope, and avoid long non-cancellable executions. Androidx provides `viewModelScope` for view models, and `LifecycleOwner#coroutineScope` extensions for UI components. For fragments, you almost always want to use the `viewLifecycleOwner` as it is properly scoped to fragment views which are sometimes shorter lived than the fragment instance itself

When using presenters, it's very easy to create a coroutine scope similar to the ones provides for view models. Note however that they will then have to be manually cancelled when the presenter is going away.
```kotlin
private val presenterScope = CoroutineScope(SupervisorJob() + AppDispatchers.main)
```

Use `presenterScope.cancelChildren` when detaching from the view momentarily to avoid cancelling the scope prematurely


### Dispatchers
Avoid using `Dispatchers.x` directly, at least for now. Kotlinx.coroutines team plans to work on providing [better testing APIs](https://github.com/Kotlin/kotlinx.coroutines/issues/1365) in future. In any case, `AppDispatchers.x` from `pandora-coroutines` should be used instead

During unit testing, `CoroutineTestRule` SHOULD be used. It will properly override the `AppDispatchers` references so that test code can be executed in a deterministic fashion. For other testing needs, please refer to the official [kotlinx-coroutines-test](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/) documentation for more details


### Exception Handling
Avoid catching **all** throwables when using coroutines, or even in general. In cases where this is unavoidable, there are a few things to note especially when using coroutines. All CancellationExceptions that may have been caught MUST be actively re-thrown as this is the signal for the coroutine machinery to properly clean itself up.

Consider using [CoroutineExceptionHandler](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-exception-handler/index.html) whenever possible. This will help keep the code clean and avoid lots of try/catch nesting and associated subtleties. Because of the high uncertainty of execution context that comes with using CoroutineExceptionHandler directly, `pandora-coroutines` provides a `CoroutineScope.exceptionHandler` extension that is particularly optimized for UI centric exception handling. E.g to show some feedback on the UI after an exception has been caught
```kotlin
val exceptionHandler = scope.exceptionHandler {
  throwable ->
  transformErrorAndNotifyUI(throwable)
}
scope.launch(exceptionHandler) {
  someMethodThatCanThrowException()
}
```

When a try/catch is absolutely required, please use with caution. Prefer to catch only very specific exceptions when possible, otherwise consider the exception handler alternative described above first. Try/finally on the other hand is absolutely fine as it does not attempt to catch any exception

If it turns out that **all** exceptions must be caught, either via a try/catch or with runCatching function from the standard library, Kotlinx.coroutines provides an `ensureActive` suspend function that SHOULD be called before processing such exceptions. This ensures that the cancellation signal, if any, will be properly delivered to the coroutine machinery
```kotlin
try {
  someMethodThatCanThrowException()
} catch(t: Throwable) {
  ensureActive()
  // continue handling t from here
}
```

When using flows, a built-in [catch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/catch.html) operator is available to properly handle exceptions without interfering with the coroutine machinery. Use them as much as is needed
```kotlin
someFlow
  .catch { e -> doSomethingWithException(e) }
  .collect { ... }
```


### Migrating from Rx
 Use this simple mental model when migrating from an Rx codebase
```kotlin
fun(): Single<T> => suspend fun(): T
fun(): Maybe<T> => suspend fun(): T?
fun(): Completable => suspend fun()
fun(): Observable<T>, Flowable<T> => fun(): Flow<T>

fun(): Subject<T>, Processor<T>, Relay<T> => fun(): StateFlow<T>, SharedFlow<T>
```
This migration can happen incrementally, so it doesn't have to be all or nothing. Kotlinx.coroutines provides a [kotlinx-coroutines-rx2](https://github.com/Kotlin/kotlinx.coroutines/tree/master/reactive/kotlinx-coroutines-rx2) bridge that can be used to interop between Rx and coroutine code during the migration process

### Transforming callbacks
When transforming a callback based API to coroutines, there are a few possibilities.

For simple one-off calls, use [suspendCancellableCoroutine](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/suspend-cancellable-coroutine.html) to transform the callback to a suspend function.
For example
```kotlin
// Given
fun login(request: Request, onSuccess: (Response) -> Unit, onError: (Throwable) -> Unit) {
  ...
}

// Transform to suspend fun
suspend fun login(request: Request): Response {
  return suspendCancellableCoroutine { continuation ->
    login(
      request,
      onSuccess = { response -> continuation.resume(response) },
      onError = { throwable -> continuation.resumeWithException(throwable) }
    )
  }
}
```

Otherwise, for continuous stream of data, flow based APIs can be utilized, using [callbackFlow](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/callback-flow.html) for cold streams, or [SharedFlow](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html) for hot streams accordingly

--

For further information, questions, and/or improvements, feel free to reach out on [#pd-tech-android](https://deliveryhero.slack.com/archives/G4TGV5S7L) Slack channel
