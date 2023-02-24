## Serialization

Our recommended tool of serialization is [Kotlinx-Serialization](https://github.com/Kotlin/kotlinx.serialization) (KxS). The reasoning behind the choice of this tool can be found in this [RFC](https://github.com/deliveryhero/pd-box/issues/882).

Below you will find the steps in order to have it integrated with your feature:

## 1. Adding dependencies

In order to have the integration with KxS you should add the serialization plugin and the json serialization library.

```gradle
plugins {
    id("kotlinx-serialization")
}

dependencies {
   implementation(libs.kotlinxSerializationJson)
   implementation(libs.retrofit)
   implementation(projects.networkApi)
   implementation(libs.pandoraApi) // we will need it to set the converter factory.
}
```

## 2. Integrating with Retrofit

In order to deserialize your backend responses with KxS, you need to install a Kxs `Converter.Factory` in your `Retrofit`. For
that you can follow the guidelines below on the function responsible for creating your feature service.

It is important to note that our Retrofit objects are being created in `metalcore` and they come with built-in GSON factories (this was the previous tool we supported). In order to override those factories we have to set our KxS converter factory by using the metalcore helper method `fun Retrofit.Builder.setConverterFactory(factory)`:

```kotlin

@Provides
@Reusable
fun provideYourApi(
    serviceProvider: PandoraServiceProvider,
    @KotlinSerializationApi converterFactory: Converter.Factory,
): YourApiInterface {
    return serviceProvider.createService {
        setConverterFactory(converterFactory)
    }
}
```

You can also configure the `Json` object to provide different properties such as:
- `ignoreUnknownKeys`
  - Specifies whether encounters of unknown properties in the input JSON should be ignored instead of throwing SerializationException)
- `isLenient`
  - a lenient parser becomes even more permissive to invalid value in the input, replacing them with defaults)
- `coerceInputValues`
  - Enables coercing incorrect JSON values to the default property value in the following cases:
     - JSON value is null but property type is non-nullable.
     - Property type is an enum type, but JSON value contains unknown enum member.

For example, you could declare that `Json()` like this:

```kotlin
val json = Json {
    coerceInputValues = true
    ignoreUnknownKeys = true
    isLenient = true
}

val serializationConverterFactory = json.asConverterFactory("application/json".toMediaType())
```


## 3. Declare your classes

Let's use a small example to understand how we map a JSON structure to Kotlin using KxS. For that we would have the following json:

```json
{
  "name": "Androider",
  "age": 55,
  "user_hobbies": [
    "Coding", "Reading"
  ]
}
```

For that we would start off by declaring a class to represent it:

```kotlin
data class(
  val name: String,
  val age: Int,
  val hobbies: List<String>
)
```

**Generating the Serializer**

So that KxS can generate the serializer for the class, you need to annotate it with the `@Serializable`.

```kotlin
@Serializable
data class(
  val name: String,
  val age: Int,
  val hobbies: List<String>
)
```

**Using the JSON field original name**

In some situations we will want to model the name of the field in our class differently than the one used in the `json`. In our example, the `json` uses `user_hobbies`, but our class uses `hobbies`.

To accommodate that, we can use the `@SerialName` annotation in order to associate that class field with the json field.

```kotlin
@Serializable
data class(
  val name: String,
  val age: Int,
  @SerialName("user_hobbies")
  val hobbies: List<String>
)
```

**Define default values**

In some cases, we also want to have default values in case the value is not present. For that we can simply use kotlin default values and
make sure that our converter factory is configured with `Json.isLenient/coerceInputValues` enabled.

For our example, we will assume that `user_hobbies` is an optional field, and that we want to default to an empty list:

```kotlin
@Serializable
data class(
  val name: String,
  val age: Int,
  @SerialName("user_hobbies")
  val hobbies: List<String> = emptyList()
)
```
