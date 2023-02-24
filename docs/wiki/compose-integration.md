## Compose Integration

### Adding dependencies

In order to develop with Jetpack Compose and Bento in your module, add the following configuration to your `build.gradle.kts`:

```kotlin
android {
    buildFeatures.compose = true
}

dependencies {
    implementation(libs.bundles.androidxCompose)
    implementation(projects.composeFoundation)
    implementation(libs.bundles.bentoCompose)

    kotlinCompilerPluginClasspath(libs.androidxComposeCompiler)
}
```

### Loading your Composable in an `Activity`

In order to render your Composable inside your `Activity`, you should use the `setBentoContent` method. This method takes a `content: @Composable () -> Unit` and will render your Composable under the theme of Bento.
This is the only approach to take in order to have the correct theming and access to Bento specific definitions such as colors, spacings, or corner radius.
You also need to apply `BentoTheme` or one of its descendants to your activity in order to use multicolored drawables and correct Modals appearance.
You can do that in the manifest file or programmatically via `setTheme()`.

```kotlin
class FeatureActivity: AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setBentoContent {
            /* your awesome compose screen */
        }
    }
}
```

### Loading your Composable in a `Fragment`

In order to render your Composable inside your `Fragment`, you should use the `createBentoView` method. This method takes a `content: @Composable () -> Unit` and will render your Composable under the theme of Bento. 
This is the only approach to take in order to have the correct theming and access to Bento specific definitions such as colors, spacings, or corner radius.

```kotlin
class FeatureFragment: Fragment() {

   override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View =
       createBentoView {
          MyFeatureComposable()
       }
}
```

### Showing Composable Modal as a `DialogFragment`

In order to render your Composable modal inside `DialogFragment` you should use one of our helpers: `BentoDialogFragment`. It already includes all the necessary settings.

```kotlin
class ComposeModal : BentoDialogFragment() {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding.container.setBentoContent { 
            Modal {
                ModalContent {
                    /* your awesome composable modal */
                }
            } 
        }
    }
}
```

### Showing Composable bottom sheet as a `DialogFragment`

In order to render your Composable bottom sheet inside `DialogFragment` you should use one of our helpers: `BentoBottomSheetDialogFragment`. It already includes all the necessary settings.

```kotlin
class ComposeBottomSheet : BentoBottomSheetDialogFragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        /* some logic */
        return createBentoView {
            /* some logic */
            FragmentContent { sheetState, scope ->
                ModalBottomSheetLayout(
                    sheetContent = {
                        BottomSheetContent {
                            /* your awesome composable bottom sheet */
                        }
                    },
                    content = { /* empty, since we need only the bottom sheet */ }
                )
            }
        }
    }
}
```

## Accessing the `StringLocalizer`

The `compose-foundation` provides a way to access a `StringLocalizer` anywhere inside the tree of `Composable`. For that you can simply call the method `stringLocalizer()` to obtain a reference to it.

```kotlin
@Composable
fun myComposable() {
   val stringLocalizer = stringLocalizer()
   Text(text = stringLocalizer.getText(TranslationKey.MY_KEY))
}
```

## Using Preview

In order to use `Preview` with our Bento theme, you should wrap your composables inside a `BentoPreview`.
It has a predefined `brandConfiguration` of Foodpanda and a default `StringLocalizer` (it returns the same key it was provided).
You can override these values.

```kotlin
@Preview
@Composable
fun MyPreview() {
    BentoPreview {
        MyComposable()
    }
}
```
