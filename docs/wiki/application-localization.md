## All apps work with remote and local translations.

**Remote translations** are managed with [Webtranslateit Tool](https://webtranslateit.com/en)
where content team and product squad manager fill with translation keys and translated values.

**Local translations** xml files are automatically created and filled by executing the `updateTranslations` script located on the root of the project.
On Bitrise CI/CD tool there is a workflow `update_translations` defined to run this script when something is merged to development branch or any release branches in order to keep the main branches up to date with latest translation changes.
For more info on how this script works please visit Readme file of the following repository:
https://github.com/deliveryhero/pd-mobile-utils
Inside of this repository `translations.php` file contains the main logic for generating translation files for both Android and iOS.

## Who to contact?
Squad PMs are responsible for providing all new translation keys in the Jira ticket and adding them on the Webtranslateit tool.

## How to get keys?
All **strings.xml** and the corresponding **TranslationKeys** are now inside **translations** module. Feature modules that need translations should not create their own strings.xml and TranslationKeys class, but instead, they should depend on this module for such provision.

When developer needs the new translation keys and they are not yet available in `development` branch, the suggested way is to trigger `update_translations` workflow on `development` branch, manually on Bitrise and all new keys will be automatically pushed to `development`. The new keys will then be available for use after rebasing unto `development`.

The project may need to be rebuilt so that the annotation processor can regenerate the TranslationKeys class with all the new keys included

## Basic rules:
* PRs should not contain any translation key changes to maintain smaller PRs.
* All keys should start with `NEXTGEN_{key_name}`, Pandora naming convention.
* Existing translation keys should not be removed from the dashboard because of backward compatibility, but rather should be deprecated and create new ones for the same purpose.


Technical details for using the keys in the application code-base:
`localization` module in metalcore repository contains all the logic for translating keys to local String values.
`LocalizationManager` has two public methods `getTranslation(key)` and `addProvider(provider)`.
`addProvider` method adds different types of providers to `providers` list (`RemoteLocalizationProvider`, `LocalLocalizationProvider`) `getTranslation` method checks if available providers are capable to translate the provided key. Order of adding providers to the list is important as if the first provider can translate the key, will exit and return the translated value.
