The plug-in architecture pattern is a natural pattern for implementing product-based applications. The plug-in architecture allow us to add additional application features as plug-ins to the core application, providing extensibility as well as feature separation and isolation.

To extend the core application, you must provide a binding. For example, a typical binding is a feature `Initializer` that will be executed on `Application.onCreate`:

```kotlin
// ContributesMultibinding ensures that the plug-in point is properly connected
@ContributesMultibinding(ApplicationScope::class)
class ConfigsInitializer @Inject constructor(
    private val configManager: ConfigManager,
) : Initializer {

    fun initialize() {
        warmupConfigs()
    }
}
```

The following plug-in points are currently available for usage by the feature teams:

| Plug-in Point | Type | Key | Status | Description |
|---|---|---|---|---|
| Initializer | Set | - | ✅ | Invoked on `Application.onCreate`. |
| DhRoute | Set | - | ✅ | Invoked on app deeplink. |
| DhRedirect | Set | - | ✅ | Invoked on website deeplink |
| ApiErrorHandler | Set | - | Invoked when backend service error is encountered |
| OnLoginListener | Set | - | ❎ | Invoked on user log in. |
| OnLogoutListener | Set | - | ❎ | Invoked on user log out. |
| AnalyticsEventCallback | Set | - | ✅ | Invoked for every tracked event. |
