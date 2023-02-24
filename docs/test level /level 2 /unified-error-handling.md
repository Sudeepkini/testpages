## Introduction
This page describes our unified API to process and gracefully recover from API errors. This API was designed with a few pre-requisites in mind

#### Unified API for both old and new error response formats
While we have different kinds of API errors, there are 2 standard structures for these responses and this API tries to make it transparent as much as possible. As a result, custom handlers, in most cases, do not require any knowledge of the exact structure of these responses. However, situations where the responses come with extra metadata may still require that such differences are properly taken into account

#### Decentralization of feature specific errors
Prior to this API, we had a huge god class that knows about all the different kinds of possible API errors and knows how to transform an error into a corresponding exception. This has lots of disadvantages and also means that everyone had to live with this baggage of knowledge. The unified solution on the other hand keeps the core APIs very simple and minimalistic, and exposes hooks such that each independent features may register their own custom handlers

## Architecture
<img src="https://github.com/deliveryhero/pd-mob-b2c-android/raw/development/wiki-images/error-handler-architecture.png" />

## Custom Error Handlers
Without any handlers registered, `ApiErrorProcessor` will simply return the base `ApiException` with very basic details which may be sufficient for some scenarios. There are cases however, where we may need to return a customized subclass of `ApiException` and/or perhaps retrieve more content from the error metadata. In such case, an implementation of `ApiErrorHandler` should be registered to the global `ApiErrorProcessor`

A custom implementation for generic `ConstraintViolationErrorHandler` is already available and may be installed into ApiErrorProcessor to catch all matching cases. However, feature owners are encouraged to create and register custom handlers whenever this can help them create richer experiences for users.

### Exception materialization
The order in which handlers are registered does not matter as the processor will always go through the whole list to retrieve output from all matching handlers. However, in a situation where this results in more than 1 item, a composite exception containing a list of all the exceptions will be returned instead. This means attempting to catch one of those exceptions will fail.

To materialize the correct exception, the stream needs to be explicitly unwrapped. This can be done using the `unwrapException` extension defined on all Rx streams (Single, Maybe, Observable, Flowable and Completable), as well as Coroutine Flow. Alternatively, the throwable can also be directly materialized using `Throwable.forType` extension

A recommended approach is to define a single top level Exception (that extends from ApiException) in each feature module. And optionally implement a simple `unwrap()` extension for Rx/Coroutine streams accordingly. Then all subsequent types extending from this base Exception implicitly reaps the benefit. This way, use sites can simply include an `unwrap()` operator in the chain, and everything works as expected

For example
```kotlin
internal open class MyFeatureException(errorInfo: ErrorInfo): ApiException(errorInfo)

internal class MySpecificFeatureException(
    errorInfo: ErrorInfo,
    val someExtraValue: String
) : MyFeatureException(errorInfo)

private val EXCEPTIONS = listOf(MyFeatureException::class)

internal fun Single.unwrap() = unwrapException(EXCEPTIONS)
internal fun Flow.unwrap() = unwrapException(EXCEPTIONS)

// Use-site
someSingle
  .subscribeOn(someScheduler)
  .observeOn(mainScheduler)
  .unwrap()
  .subscribe(onSuccess, onError)
```


For more information, please refer to code documentation on corresponding classes/interfaces
