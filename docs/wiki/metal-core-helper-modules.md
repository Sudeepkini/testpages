## MVVM commons module

### How to use

This module contains the view model factory and the annotation `ViewModelKey`, so you can use Dagger to inject the viewmodels.
In order to use the `ViewModelFactory` you need to provide it from some module like this:
```
@Module(includes = [ViewModelModule::class])
internal class SomeModule {

    @Singleton
    @Provides
    fun providesViewModelFactory(viewModels: Provider<Map<Class<out ViewModel>, Provider<ViewModel>>>): ViewModelProvider.Factory {
        return ViewModelFactory(viewModels.get())
    }
}
```
Some important things to note here are:
* `@Module` includes the module which provides the view models
* `@Provides` will use the provider to create the `ViewModelFactory` we cannot use `@Inject` constructor or provide it from metalcore, otherwise we'll have duplicated generated classes

There are also a couple of helper classes like: `Resource` and `ResourceState` which can be useful to create a simple event based stream

## Image loading module

### Introduction

This image loading module will just be an abstraction to Glide for now. But it's been written in a way that if we wanted to change it in the future there will be minimal changes to the clients.
We are abstracting it because we have`ImageUrlBuilder` which is supposed to generate links based on the device size and internet quality. So instead of using it everywhere in the project now it will be implicitly used by everyone who is using these extensions functions.

**Warning:** There are some places in the app where the BE would return the URL with the query param `width=%s`, please make sure to communicate that this is no longer needed. As the `ImageUrlBuilder` already adds this for you

### How does it work?

It has a few extension functions on `View` and `ImageView` based on the most popular usages we had on the app. To use it just call the method `loadImage` and give your url. The last parameter is a lambda where you can call all of Glides methods to configure how to get the image.
You can also configure the quality of the image you want, the options are: `WithWidth`, `WithHeight`, `OptimizedWidth`, `FullWidth` and `SmallestWidth`, the default being `OptimizedWidth` and that takes into account the internet quality and device width.
There are also extension methods for `preLoad` and `clear`.
In case you need to load an image into a custom view target, there is an extension function which can get three callbacks for `preLoad`, `success` and `error`.
One downside to this abstraction is that we can only get a `Drawable` (`GifDrawable` works too), so if you need a `Bitmap` you can use `BitmapUtils` from `image-processing` to turn the `Drawable` into a `Bitmap`

**Warning:** Make sure you don't use any imports from Glide. In the near future there will also be a `Lint` rule which will fail if you're using `Glide` imports in the project

**Note:** If you use this module, you will also need to explicitly declare `Glide` as a dependency. This is something that might be fixed in later versions.
