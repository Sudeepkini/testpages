# Dependency Injection

Dependency Injection (DI) is a design pattern to instantiate loosely coupled classes. It reduces the dependency coupling between modules by providing a facility to construct instances of classes with their dependencies injected, and manage their lifetime based on the configuration of the given component.

We are using *DI* to decouple our modules and encourage unit testing best practices at the same time. The DI frameworks we are using in our app are:
- [Dagger 2 by Google](https://github.com/google/dagger).
- [Anvil by Square](https://github.com/square/anvil).

Our goals are:
- To simplify Dagger-related infrastructure inside our application.
- To allow our project to modularize further without any custom configuration nor module aggregations.
- To create a standard set of components and scopes to ease setup, readability/understanding, and code sharing between modules.

Heads-up: our Dependency Injection framework is called **[Whetstone](https://github.com/deliveryhero/whetstone)**.

## Table of Contents

1. [Gradle Setup](#gradle-setup)
2. [Guide](#guide)
3. [Testing](#testing)
4. [Custom Components](#custom-components)
5. [Custom Injection Point](#custom-injection-point)

### Gradle Setup

To use DI, you need to add the following build dependencies to your module `build.gradle` file.

```groovy
plugins {
  // ...
    id("com.deliveryhero.whetstone")
}

whetstone {
    // Note you MUST remove `kapt(libs.daggerCompiler)`
    // Note if you have to include compose for dependency or workManager
    addOns {
        compose.set(true)
        workManager.set(true)
    }
}
```

### Guide

Unlike traditional Dagger, you do not need to define or instantiate Dagger components directly. Instead, we offer predefined components that are generated for you. `injection` comes with a built-in set of components (and corresponding scope annotations) that are automatically integrated to the Pandora App. As normal, a binding in a child component can have dependencies on any binding in an ancestor component.

#### Component Lifecycle

Component lifetimes are generally bounded by the creation and destruction of a corresponding instance of an important event. The table below lists the scope annotation and bounded lifetime for each component.

| Component | Scope | Created At | Destroyed At |
| ----------- | -------- | -------- | -------- |
| ApplicationComponent | @ApplicationScope | Application#onCreate | Application#onDestroy |
| ActivityComponent | @ActivityScope | Activity#onCreate | Activity#onDestroy |
| FragmentComponent | @FragmentScope | FragmentFactory#instantiate | Fragment#onDestroy |
| ViewModelComponent | @ViewModelScope | ViewModelProvider.Factory#create | ViewModel#onCleared |

#### Basic Usage

The best way to understand it is by a simple example of how to use it. The following code uses DI to implement loose coupling:

```kotlin
interface Repository

@ContributesBinding(ApplicationScope::class)
class RepositoryImpl @Inject constructor() : Repository

class MyViewModel @Inject constructor(
    private val repository: Repository
) : ViewModel()
```

This is what we know when we have a look a little closer to this class:

- `MyViewModel` needs an *interface* `Repository` to work.
- `MyViewModel` doesn't know which class is implementing the *interface* `Repository` and how it was initialized.
- If we would want to test `MyViewModel`, we don't need the `RepositoryImpl` that implements the interface `Repository`, we just need a *fake* of this interface.

#### Contributing with Constructor Injector

The recommended way to create dependencies is using constructor injection. This is the simplest way to use DI in our project.

```kotlin
class SessionHeaderInterceptor @Inject constructor(
    private val requestSession: RequestSession
) : Interceptor
```

#### Contributing with a Binding

The second recommended approach is using Constructor Injection and providing a binding to an interface:

```kotlin
interface Repository

@ContributesBinding(ApplicationScope::class)
class RepositoryImpl @Inject constructor() : Repository
```

Remember that `ContributesBinding` does not provide a scope for the object itself. Therefore, if we want to have only one repository per application instance we would need to do add the scope:

```kotlin
@ContributesBinding(ApplicationScope::class)
@SingleIn(ApplicationScope::class) // scope annotation
class RepositoryImpl @Inject constructor() : Repository
```

#### Contributing with External Dependencies

Sometimes, we need to create modules for dependencies we do not own. This is the case when integrating with third-party libraries like Retrofit or OkHttp.
For these cases, we must create a `@Module` by our own. For example:

```kotlin
@ContributesTo(ApplicationScope::class) // We want to install this module into ApplicationComponent.
@Module
object AppModule {

    @Provides
    @SingleIn(ApplicationScope::class) // Single instance in ApplicationScope.
    @ForScope(ApplicationScope::class) // Qualifier to identify an Application context.
    fun provideAppContext(
        application: Application,
    ): Context {
        return application
    }
}
```

This is the last recommended option, it is designed to be used only when integrating with systems we do not have control over the construction process (e.g., Retrofit, Android Framework classes, OkHttp, etc).

#### Contributing with the Android Framework

One of Android's greatest challenges is that classes coupled to the OS can be created, destroyed and recreated under a variety of circumstances - everything using the no-args default constructor.

Luckily, Android Jetpack comes with many utilities to workaround this limitation and we provide integration with Jetpack! To contribute with a ViewModel or Fragment, you must do some extra configuration showed below.

##### ViewModels

To let Android use our DI to create a ViewModel you must do the following:

```kotlin
@ContributesViewModel
class MyViewModel @Inject constructor(
    private val savedStateHandle: SavedStateHandle,
): ViewModel()
```

This will add the ViewModel to `ViewModelProvider.Factory` for usage.

**Important:** A ViewModel should **NEVER** be scoped. The Android Framework controls the Lifecycle of **ALL** ViewModels.

##### Fragments

To let Android use our DI to create a Fragment you must do the following:

```kotlin
@ContributesFragment
class MyFragment @Inject constructor(
    private val someDependency: SomeDependency,
): Fragment() {

    // Get the contributed ViewModel
    val viewModel by injectedViewModel<MyViewModel>()
}
```

**Important:** A Fragment should **NEVER** be scoped. The Android Framework controls the Lifecycle of **ALL** Fragments.

##### Connecting to an Activity

To connect an `Activity` to the Dependency Injection system, you must directly opt-in:

```kotlin
internal class SurveyActivity : AppCompatActivity() {

    // Get the contributed ViewModel
    val viewModel by injectedViewModel<MyViewModel>()

    override fun onCreate(savedInstanceState: Bundle?) {
        // `inject` must be called BEFORE `super.onCreate`.
        Whetstone.inject(this)
        super.onCreate(savedInstanceState)
        // Set up Navigation Component or inflate the main fragment.
    }
}
```

For custom injection in an `Activity`, see "Custom Injection" topic.

#### Contributes To

| Contributes to | Component |
| ----------- | -------- |
| **@ContributesTo(AppScope::class)** | Module is installed in `ApplicationComponent`. |
| **@ContributesTo(ActivityScope::class)** | Module is installed in `ActivityComponent`. |
| **@ContributesTo(FragmentScope::class)** | Module is installed in `FragmentComponent`. |
| **@ContributesTo(ViewModelScope::class)** | Module is installed in `ViewModelComponent`. |

#### Scoped By

| Scoped by | Resolved as | Examples | Docs |
| ----------- | -------- | -------- | -------- |
| **@Reusable** | Indicates that the object returned by a binding may be (but might not be) reused | For objects where the state does not matter, prefer `Reusable` over `ApplicationScope` | [Reference](https://dagger.dev/dev-guide/) |
| **@ApplicationScope** | Identifies a type that the injector only instantiates once | Custom Scope |
| **@ActivityScope** | Identifies a type that the injector only instantiates once per Activity | Custom Scope |
| **@FragmentScope** | Identifies a type that the injector only instantiates once per Fragment | Custom Scope |
| **@ViewModelScope** | Identifies a type that the injector only instantiates once per ViewModel | Custom Scope |
| **unscoped** | Unidentified objects will always return a new instance | All Fragments and ViewModels of the app are registered without a scope: the Android Framework is responsible to create and manage their Lifecycle | [Reference](https://dagger.dev/dev-guide/) |

To scope by an instance, you must use the annotation `@SingleIn`.

**WARNING:** A common misconception is that all bindings declared in a `@Module` with `@Contributes*` will be scoped to the component it is contributed to. However, this isn’t true. Only bindings declarations annotated with a scope annotation will be scoped. For example:

```kotlin
@Module
@ContributesTo(ApplicationScope::class) // contributes to ApplicationComponent
object MyRepositoryModule {

    @Provides
    @SingleIn(ApplicationScope::class) // single instance per ApplicationComponent
    fun provideMyRepository(): MyRepository = TODO("not implemented")
}
```

##### When to scope?

Scoping has a cost on both the generated code size and its runtime performance so use scoping sparingly. The general rule for determining if a binding should be scoped is to only scope the binding if it’s required for the correctness of the code. If you think a binding should be scoped for purely performance reasons, first verify that the performance is an issue, and if it is consider using `@Reusable` instead of a component scope.

Last but not least, Android ViewModel is a scope per itself: it is created by the Android system and it will live until the current Lifecycle Owner gets terminated - Dagger has no control on when it will happen. Additionally, you can use it to scope a state and avoid unnecessary scopes.

### Testing

The structure presented here let you define constructor injector for all your objects and therefore, the recommended approach is use the constructor to replace the dependencies you need in your tests and manually create your SUT (System Under Test).

### Custom Components

Our Dependency Injection has predefined components for Android that are managed for you. However, there may be situations where the standard components do not match the object lifetimes or needs of a particular feature. In these cases, you may want a custom component. However, before creating a custom component, consider if you really need one as not every place where you can logically add a custom component deserves one.

For example, consider a background task. The task has a reasonably well-defined lifetime that could make sense for a scope. Also, if there were a request object for that task, binding that into Dagger may save some work passing that around as a parameter. However, for most background tasks, a component really isn’t necessary and only adds complexity where simply passing a couple objects on the call stack is simpler and sufficient. Before committing to adding a custom component, consider the following drawbacks.

Adding a custom component has the following drawbacks:
- Each component/scope adds cognitive overhead.
- They can complicate the graph with combinatorics (e.g. if the component targets a `View` which conceptually can be parent of any Android component which can be a child of an `Activity` or `Fragment`, two components likely need to be added).
- Components can have only one parent. The component hierarchy can’t form a diamond. Creating more components increases the likelihood of getting into a situation where a diamond dependency is needed. Unfortunately, there is no good solution to this diamond problem and it can be difficult to predict and avoid.
- Custom components work against standardization. The more custom components are used, the harder it is for shared libraries.

With those in mind, these are some criteria you should use for deciding if a custom component is needed:

- Dynamic Features.
- The component has a well-defined lifetime associated with it (e.g., session).
- The concept of the component is well-understood and widely applicable. Predefined components are global to the app so the concepts should be applicable everywhere. Being globally understood also combats some of the issues with cognitive overhead.

#### Adding a custom component

To create a custom component, create an interface with all dependencies you need to retrieve from our main graph, with the suffix `Dependencies`. For example:

```kotlin
@ContributesTo(ApplicationScope::class)
interface CustomComponentDependencies {
    fun getRetrofit(): Retrofit
    fun getOkHttpClient(): OkHttpClient
}
```

And now, to use your component you can access the dependencies by `Whetstone.forApplication` and connect it to your component normally:

```
class CustomActivity: AppCompatActivity() {

    // At any place, do:
    override fun onCreate(savedInstanceState: Bundle?) {
        val dependencies = Whetstone.forApplication<CustomComponentDependencies>(application)
        val component = CustomComponent.factory().create(dependencies)
        super.onCreate(savedInstanceState)
    }
}
```

### Custom Injection Point

A custom injection point is the boundary where you can get Dagger-provided objects from code that cannot use Dagger to inject its dependencies. It is the point where code first enters into the graph of objects managed by Dagger.

#### When do you a custom injection point?

You will need an entry point when interfacing with non-Dagger libraries or Android components that are not yet supported and need to get access to Dagger objects.

In general though, most entry points will be at Android instantiated locations like an `Activity`. `Whetstone.inject` is a specialized tool to handle the definition of injection entry point. Since this is already handled specially for those Android classes, for the current doc, we’ll assume the custom injection point is needed in some a legacy class.

#### How to use an entry point?

To create an injection point point, define an interface with the class name and the suffix `Injector`, include a a single `inject` method, and mark the interface with `@ContributesTo(SCOPE)`.

```kotlin
@ContributesTo(ActivityScope::class)
interface CustomActivityInjector {
    fun inject(target: CustomActivity)
}
```

To access an `Injector`, use the `Whetstone` class passing as a parameter the interface we defined above:

```kotlin
class CustomActivity: AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        Whetstone.forActivity<CustomActivityInjector>(this)
            .inject(target = this)
        super.onCreate(savedInstanceState)
    }
}
```
