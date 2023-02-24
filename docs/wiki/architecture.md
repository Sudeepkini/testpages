## Architecture details
We follow [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) by Robert C. Martin (Uncle Bob) which is composed of 3 layers:
1. **Presentation Layer**: In this layer we have views and presentation logic implemented in presenters or view models. Mainly we follow `MVP` architecture pattern, however we have few places where we have implemented `MVVM`. Mapping to View Entities should be done in this layer and off main thread.
Model Entity naming: `*UiModel` (e.g. VendorUiModel)
2. **Domain Layer**: This layer contains abstractions for `Repositories`, Common Entities and `UseCases`.
    In [Metalcore repository](https://github.com/deliveryhero/pd-metalcore-android) inside of `commons` module we have base classes for `UseCases` which each `UseCase` should extends from.
Model Entity naming: `*` (e.g. Vendor)

**Base UseCases**

```
interface CoroutineUseCase<Params, Result> {
    suspend fun run(params: Params): Result
}
```

```
interface RxUseCase<Params, Result> {
    fun run(params: Params? = null): Observable<Result>
}
```

3. **Data Layer**: This layer contains implementation for Repositories abstraction from domain layer. Each `Repository` should depend on `RemoteDataSource` and/or `LocalDataSource`. `RemoteDataSource` uses `ApiClients` to connect from remote and `LocalDataSource` depends on `MemoryCache` and/or `DiskCache` interfaces located in `commons` module. Mapping to Domain Entities should be done in this layer.
Model Entity naming: `*ApiModel` (e.g. VendorApiModel)
