# Image loading library

## Why do we have a wrapper around image loading library?

There are two main reasons why we have this wrapper library for image loading

1. We don't want to have a strong dependency on any image loading library
2. We can apply optimization before requesting images and have more efficient image loading throughout the app

## What kind of optimizations are happening with this library?

Pandora image service has a very useful functionality for resizing images before we request them. This is fairly straightforward, we can add a query param to let the service know what size image we want.
For ease of use, all of this is done automatically by the library.

### How do we choose the best size?

Behind the scenes we get the size of the view container which is showing the image and also check the network quality to determine the best size of the image we want to download.

### Can I opt out of this optimization and choose a static size for my images?

In some cases it makes sense to use the highest quality image or even a lower one (for example thumbnails). In these cases you can pass one of the other available options (check [Gist](#Sample usage))

## Sample usage

Here's a small [Gist](https://gist.github.com/gent-ahmeti/24722bd83e095d4755dafeccc6f4d575) which shows some basics on how to use `image-loading` from `metalcore`

## Image loading with Compose

Now that Compose has become stable, and we're planning to start using it more and more on our project. We have created custom image loading to take advantage of the optimizations we have with our image loading library
This library is part of the b2c project, so using it is as simple as depending on a local project
```kotlin
implementation(projects.composeImageLoading)
```

### Drawbacks

* Currently we cannot use cross fade transition for images

## Sample usage

The public api of the Compose and non-Compose implementation are very similar, since both use Glide behind the scences. You can check some sample usage in this [Gist](https://gist.github.com/fevziomurtekin/f3814470e181ae989981a0c558b0fdd9)
