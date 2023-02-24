## Motivation


When covering a TestRail test case with an E2E test, the test case status in TestRail gets updated after the test execution is done during the release cycle. Some test cases can be fully covered with a unit or/and integration tests. Those tests run on JVM and are way faster than E2E tests. To encourage people to write more of those tests when trying to cover a test case, we needed to make it possible to update TestRail tests cases for those tests too. That's why we have added our own unit/integration tests reports that will be shared with our TestRail service which will update the test cases in TestRail for the reported tests.


## How to use this new feature?

There are only three things you need to add to your test class and methods to report your tests to TestRail correctly. That's all you need to do, and everything else will work out of the box.



* Add `@TestOwner(TestOwners.name)` to your testing class passing the squad name that owns this test as a parameter to the annotation. You can find all squad names inside `TestOwners.kt`
* Add the `@TestRail(ids)` annotation to each testing method passing the id of your TestRail test case as a parameter to the annotation. You can pass multiple ids as well.
* Add one of the `@UnitTes` or `@IntegrationTest` annotations to your test. This will help us to have some statistics about how many tests we have for each category. In the future, we can execute only one category if needed in some cases.

Here's an example of a unit test


```
@TestOwner(TestOwners.SQUAD_NAME)
class UnitTestExample {

   @TestRail("11111", "22222")
   @UnitTest
   @Test
   fun `Test Method Example 1`()  { //Testing Code }
```

## How Does it work

We have added a new module to the B2C project called `:ta-foundation-unit`  \
This module contains all the code we need to generate our report right after the test execution is finished using a JUnit listener. Since we have many modules in the project containing some unit/integration tests, we added our new module ":ta-foundation-unit" as a dependency to all of them using Gradle. Any new module added to the project will also get this dependency automatically. All those modules will also have access to all the libraries needed to start writing unit/integration tests immediately.


## Where is the report generated?

After the test execution, the report is generated as a JSON file and saved to this folder `/tmp/pandora/unitTests` \
\
If we run all unit/integration tests in our project, the above folder will contain a JSON file for each module. This JSON file contains a lot of information about each test like test id, test case name, test class name, TestRail id, Bitreise build number, Bitrise branch name, etc

All this information is used by the TestRail service to update the test case status in TestRai.

Here's what the file looks like


```
{
       "id": "Vs6yJcnn941660764161164",
       "date": "2022-08-17",
       "test_case": "Test Method Example 1",
       "test_class": "com.deliveryhero.pandora.test.tests.UnitTestExample",
       "test_case_owner": "SQUAD_NAME",
       "platform": "ANDROID",
       "build_no": "161961",
       "timestamp": "12:23:20.537",
       "test_status": "PASSED",
       "testrail_id": "11111;22222",
       "git_branch_name": "development-regression",
       "workflow_name": "Regression-Tribe-Based",
},
```



## How do we consume these reports?

We have a workflow in Bitrise called Trigger-Unit-Tests-And-Upload-BigQuery-Logs  \
First, this workflow will build all the unit tests for all the modules in the app. We have disabled the build cache for this step to ensure all tests will run no matter what so we can generate the BigQuery reports.  \
\
Once all the tests are finished, and reports are available, we iterate over all the reports in `/tmp/pandora/unitTests` and upload them to a BigQuery table using a shell script. \
\
If the branch of the build were a release branch, we would make an API call to our TestRail service which will process the data of the reports and update the TestRail tests cases with the test status


## When is this workflow executed?

We execute this workflow during our release and regression cycle. TestRail service will be called only during the release cycle. This way we ensure all test cases reported from our unit/integration tests are reflected in the test run in TestRail.


## Any questions or concerns?

You can reach out to us by writing to our Slack channel _#pd-fd-automation_infra_ to get more support on this topic.
