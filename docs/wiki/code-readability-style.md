# Code Readability
https://docs.google.com/presentation/d/1r-oeBfb1GULMa0MHzRW3qgDqzr7ytJcqRZUiF6SO2Ac/edit?usp=sharing

Here is a [video](https://drive.google.com/file/d/1B9_aFjs-LvwKwy9KYhfxPSYYkSsXHSGj/view?usp=sharing) of this presentation with a small Q&A session at the end, for reference

## Naming convention

### Modules

- `<module-name>-api` for a high-level artifact that only contains public APIs.
- `<module-name>` (no suffix) for a low-level artifact that may contain public APIs but is primarily intended for implementation.
- `<module-name>-testing` for an artifact intended to be used while testing usages of your module, e.g. `com.deliveryhero.coroutines:coroutines-testing` with testing utilities for the `coroutines-api` artifact. We can also take advantage of test fixtures and make this available directly from the api module
- `<module-name>-demo` for an artifact intended to showcase a demo of the feature implemented in `<module-name>`
- `<module-name>-<third-party>` for an artifact that integrates an optional third-party API surface, e.g. -proto or -rxjava2. Note that a major version is included in the sub-feature name for third-party API surfaces where the major version indicates binary compatibility (only needed for post-1.x).
- Artifacts should avoid (not use) `<module-name>-core`, `<module-name>-impl`, `<module-name>-base` or `<module-name>-common` as those names are easily over used for lack of understand in the functionality being provided.

### Packaging
- Java packages within feature modules follow the format `com.deliveryhero.<module-name>`. All classes within a feature's artifact must reside within this package, and may further subdivide into `com.deliveryhero.<module-name>.<layer>` using standard Android layers (app, widget, etc.) or layers specific to the feature.
- Api modules should use the reserved `com.deliveryhero.<module-name>.api` package structure

### Classes
* Classes, methods, and variable names should be concise and self-explanatory. No Hungarian notation in naming.
* Classes should not contain any prefixes like verbs `Get`, `Fetch` (Except when classes are use-cases)
* In case of usages of any abbreviations in the Class, method or variable names, every module should have README file with full name description for each abbreviation.

**Entity class names**

* _Foo**ApiModel**_: The `_ApiModel` suffix is used for classes that map the JSON response into a class.
* _Foo_: No prefix or suffix is used for classes that represent entities.
* _Foo**UiModel**_: The `_UiModel` suffix is used for classes that contain all the information necessary to be able to render the domain model on the screen.

**UI components**

- https://confluence.deliveryhero.com/display/GLOBAL/Android+View+Naming+Convention

# Code style

* Please import and apply in AndroidStudio preferences the code style defined in the root of `pd-mob-b2c-android` project located in `code-style/Pandora-Code-Style.xml`.
* Configure `Hard Wrap at` setting in AS `Preference` -> `Code Style` to **120** columns.
* Configure AndroidStudio to add empty line at the end of each file on save.  `Preference` -> `Editor` -> `General` and ticked `Ensure every saved file ends with a line break` on `On Save` section.
* Enable the option to use `.editorconfig` so that everyone has the same code style configurations. `Preference` -> `Editor` -> `Code Style` and make sure that `Enable EditorConfig support` is ticked

## Code style rules
* Follow SOLID principles
* Method params quantity should me maximum 5 and in case needed more please consider creating model class that will contain needed parameters.
* Methods should be separated with empty line.
* Please add empty line before each `return` statement.
* Please avoid auto formatting for whole files and classes. Format should always be performed on new changes only.
* Please avoid using the `.not()` notation, always prefer the prefix operator form `!`.
