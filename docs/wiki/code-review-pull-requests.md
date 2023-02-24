## Pull Requests Requirements

* Title of the PR should contain ticket number
  * e.g. `TICKET_NUMBER: description_text`
* Branch naming format: `{type}/{small_description}/{TICKET_NUMBER}`. Examples are:
  - `bugfix/some_description_underscore_separated/TICKET_NO`
  - `task/some_description_underscore_separated/TICKET_NO`
  - `feature/some_description_underscore_separated/TICKET_NO`.
 For type `feature`, you can merge multiple branches in it before merging to the main branch. In return [feature branches](https://martinfowler.com/bliki/FeatureBranch.html) are protected so collaborators can only push changes to them via a pull request. Test automation is skipped on PRs against feature branches as they already run on branches against the main branch. Please always create `feature` branches empty and push to remote without any additional code changes.

---
## Protected branch rules
Any branch `task` or `bugfix` opened against these protected branches will require the following rules:

- `development`  - Requires 2 approvals + code owners review + Bitrise CI check + Danger
- `release/*`    - Requires 2 approvals + code owners review + Bitrise CI check + Danger
- `feature/**/*` - Requires 2 approvals + code owners review + Danger
---

## Guidelines

* A link to Jira ticket is mandatory
* Mark yourself as a reviewer before you start reviewing code.
* When building a new feature, provide links to documentation from Confluence of how this feature is build, architecture, dependencies and flow.
* Required minimum 2 approvals to be able to merge it, however consider comments from every reviewer
* Please select assignee.
* Request review from suggested reviewers on Github (Automatic suggestion is the ones the previously did changes on that part)
* Approximately 500 max lines added or removed
* When merging PRs option `Squash and merge` should be selected except when merging release branch to development (to keep each separate commits in the history).
* Screenshots of the UI is not mandatory
* Always notify your assigned buddy for the new PR review and request review through Github.
* **Labels (keep PRs with up to date labels):**
    * `Awaiting review` - Please code review this PR
    * `Code review passed` - Code review finalised
    * `DO NOT REVIEW (YET)` - Do not start to code review PR marked with this label
    * `Don't merge | WIP` - This PR should not be merged while is marked with this label
    * `Feature Branch [Don't Review]` - Marks that this PR is for feature branch. Multiple branches will be merged into this one and all separate PRs opened against feature branch will be code reviewed but not the feature branch
    * `Needs rebasing!` - This branch needs to be rebased maybe due to conflicts with the base branch
    * `Obsolete` - No longer used, could be closed in the near future
    * `Please check comments` - Owner of the PR should check and act on the review comments
    * `QA Passed` - PR should be marked with this label when the Jira ticket pass QA
    * `READY FOR QA` - Marks the PR that code review passed and is ready for QA. Important: This label triggers github webhook that will trigger build on Bitrise and automatically uploads builds on hockey app for all brands and after that comments on the Jira ticket with app build hockey links that will be used for performing manual QA. For this ticket number in the branch name is required.
    * `RERUN FAILED UI TESTS` - Marks the PR that had Bitrise failed due to failure of some UI tests. It triggers [github webhook](https://github.com/deliveryhero/pd-mob-github-hook/pull/3) that will trigger `Rerun-Failed-UI-Tests` Bitrise workflow to rerun only failing UI tests on that PR
    * `Release Branch [Don't Review]` - PR marked with this label means that is opened for release branch and should not be code reviewed
    * `Release PR` - Means that this is opened for fixing bugs for the current ongoing release and should be prioritised for code review
    * `AUTO-CREATED` - Auto created PRs against `development` branch with cherry picked changes that were merged into the `release`/`hotfix` branches during mobile releases.
    * `NOT-REQUIRED-FOR-DEV` - Use this label on PRs with changes based on `release`/`hotfix` branches if you would like to skip to auto create PRs with the same changes against development branch.
    * `Check TODOs before merging` - We often put TODOs in our code such us for updating translations later on once the PM adds them. This label reminds us to replace things like that before we merge our PRs. Adding this label allows us to still get our PR reviewed without waiting for the proper translation keys.
    * `TA` - Means PR is purely containing TA code only.
    * `TA Reviewed` - Means PR is code reviewed by a TA Engineer & approved and now can be reviewed by Mobile Engineers.

## Code Reviews Requirements - [Best Practices](https://google.github.io/eng-practices/review/reviewer/looking-for.html), [Speed of code reviews](https://github.com/google/eng-practices/blob/master/review/reviewer/speed.md)
* [Pull Request and Code Review - Best Practises](https://confluence.deliveryhero.com/pages/viewpage.action?pageId=458210991)
* Spend 1 hour in the beginning of the day for opened PRs waiting for code reviews.
* Check for open PRs waiting for code review before you start a new task or after finishing a task.
* Prioritize PRs for code review with label `Release PR`.
* Check all repositories related to android for code reviews.
* Please reply to all comments and all people that commented **need to approve** the PR before merging it.
* Always favour suggestions with examples in comments


### **IMPORTANT notes on when to merging PRs to development**

Please follow this guidelines when deciding to include changes in `development` and still maintain quality of the apps.

https://confluence.deliveryhero.com/pages/viewpage.action?spaceKey=GLOBAL&title=In-Squad+TA+Development+for+New+Features

- UI tests are required and are important
- Each ticket that is linked to a PR which will be merged to development must to contain correct `Fix Version`. So that QA team are able to analyse on the impact of changes during regression.
- Each feature (even addition to previously released feature) should be implemented with feature flag. To be able to control it while rolling it out.
- Each squad is responsible for delivering quality of the features and with this the whole team is responsible to define how they gonna test the feature, bug, improvement, tracking…  In case the squad does not have manual QA resources and still prefers to be tested by QA team the squad itself should approach `Prince Harjai` QA team lead for assigning person to perform the test if possible (preferably plan to request for this support earlier as possible).

## Code owners

To make the code review process easier and more efficient we use the `CODEOWNERS` file to assign owners to different paths or modules so people responsible will be required to sign off on the PR before getting it merged. To do this we create Github teams and add them to the file, so whenever a new PR is created with changes to one of these files a code owner is needed for approval.

### Important note

For every new team which is created and subsequently added to the `CODEOWNERS` file please notify someone from [mobile admins](https://github.com/orgs/deliveryhero/teams/pandora-mobile-admins) otherwise your team won't be assigned as required reviewers.
