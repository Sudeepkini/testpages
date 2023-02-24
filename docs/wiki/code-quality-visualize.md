## What is this documentation for?
This document is a series of guides we have written for the Code Quality Dashboard that we use to track and improve our overall code quality of the project.
These guides are mostly targeted to people who want to know how the internals of the framework works.

## How do we visualize data?
All the metrics we collect are saved in Big Query in a specific table for each category. The categories being **Code coverage**, **Issues**, **Complexity**, **Duplication** and **Pandora issues/initiatives**.
If you want to have a closer look at the schemas and tables, here's the mapping of category to Big Query table.
| Category | Table |
|----------|-------|
| Code coverage | ios-unit-test-coverage |
| Issues | code-issues |
| Complexity | code-complexity |
| Duplication | code-duplication |
| Pandora Issues/Initiatives | pandora-issues |

Android and iOS metrics are all gathered in the same tables. We differentiate between platforms using the `platform` column.
### How do we combine all the metrics into a single dashboard?
Since the score is a measurement containing all these different metrics we have created a new table which uses SQL `JOIN` query to create a big table that contains everything in one place. This table is named `pandora-code-quality`.
This table is being populated by the Bitrise workflow. If you would like to understand more how the tables are joined and how this table is being populated check [`code-quality-pandora.sql`](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/scripts/code-quality-framework/code-quality-pandora.sql).
Having all the data in one place makes it easier to manipulate and show in Data studio.
**Note:** We won't go into detail how to use Data Studio, or why we have chosen particular charts for visualization here.
### How do we show only the latest changes?
The code quality dashboard doesn't have any time filters because it will always show the latest data we have. In order for this to work we need an agreement to what *latest change* actually means.
For the dashboard we have chosen the last date we have on the `code-complexity` table. The reason behind this is that `issues`, `duplication` and `pandora issues` can be missing from the table, in fact that is our goal all these issues shouldn't exist in any given module. This leaves us with `code complexity` and `code coverage`, since `code coverage` is updated more often than the board (coverage is updated every night, while the board only weekly) we have taken `complexity` as the baseline for the latest information.
To be as efficient as we can with Big Query and Data Studio we are using an `SQL View` to visualize the data. You can check the query and schema in Big Query by looking at `Code-Quality-Scores` table/view.
### Helpers tables/views
Some of the drill down pages in the dashboard have some unique tables with more unique joins and information.
#### Duplication
The metrics we collect for `duplication` are given in different rows and have the same `id` so we can find all files which contain the duplicates. While this makes sense for an SQL table it's hard to make sense of it visually.
In order to fix this we have created a new SQL view `duplication-files` which joins the rows into a single row and also gives you the Github links with a highlighted range to point out where the duplication starts and ends.
#### Pandora issues/initiatives
Pandora issues/initiatives are internal checks and initiatives we have, because of that it isn't always clear what exactly they mean or how we should fix them.
To solve this problem we have another table named `pandora-issues-description` which contains the `issue id` and a `description` of the issue. By `joining` this table and the reports we have from the project we show a descriptive table with `priority` and `description` of the issue. The `description` should always contain a reason and the probable fix for the issue.
