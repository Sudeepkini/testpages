## Mockingjay (Mock-Server v2)


You can now use the latest version of Mock-Server (v2), which is called Mockingjay. With Mockingjay, you can quickly run any test on the Mock-Server.

Mock-Server v1 was beneficial and solved several problems. However, it was hard for developers to either write new tests or convert current tests to use Mock-Server because of one critical reason: It takes a lot of time and effort to run one test on the Mock-Server. For more information about Mock-Server v1, see [Mock-Server](https://docs.google.com/document/d/1qQ2N9LOypljuMvfkwVcAhrrlyPZYpzzp40PhGrCaFD4/edit).

## Limitations of Mock-Server v1
- The steps needed to run one test on Mock-Server were time-consuming.
- Developers had to fetch responses from Charles for every API call triggered during the test and then manually create test scenarios on the Mock-Server dashboard with the mocked responses.
- Then, they had to wait for the auto-generated code to be merged to the development branch to use those scenarios. Or, developers had to manually create those constant names and then add the constant names to the test headers.
- If one API path changed, developers needed to update the code manually.
- If a new API call was introduced to a current feature, developers had to update both the code and the Mock-Server.
- Any changes in the sequence of the API calls required developers to update the code manually.

Because of these reasons, to run your test on Mock-Server v1, you need a lot of time. We have designed Mockingjay to resolve these issues and make it easy for you to run tests on the Mock-Server.

## Features of Mockingjay
- No need to write any code in your test function, for example, TA headers, scenarios, etc.
- For each test case, Mockingjay fetches API responses from staging on your behalf.
- Only one line of code is required to run your test on Mockingjay.
- Versioning system helps you to update test records on Mockingjay easily.

## Run Test Cases on Mockingjay
As an Android developer, follow these steps to run your test cases on Mockingjay:

1. Add the **@MockedTest(testVersion = 1, mockServerVersion = 2)** annotation to your test.
2. Run the test.

After the test successfully passes, your test will be fully mocked.


![image2](https://user-images.githubusercontent.com/7036110/182635341-673d820b-7eae-4b75-a38d-689fccc7868d.png)


## Mockingjay Test-Run Phases

When you run your tests against the staging server without adding the `@MockedTest` annotation, every API call made during the test will hit the staging server. We call this phase the Organic Run.

After you add the `@MockedTest` annotation, the test runs against the staging server but through our Mockingjay server. Mockingjay forwards every API request made during the test to the staging server and remembers the order of all APIs. Then, Mockingjay sends the responses back to your test. This phase is called the Recording Phase.

After the test run is completed, Mockingjay gets notified via an API call that is made from the testing application when the test finishes. If the test status is successful, Mockingjay caches all API responses for that specific test in the database. If you run your mocked test again, Mockingjay replies with the cached responses instead of hitting the staging server. This phase is called the Replay Phase.

## Frequently Asked Questions

### How to mock a new test correctly?


Before you send your test for code review, we recommend that your mocked test undergoes all three stages on your local machine:

- Organic Run
- Recording Phase
- Replay Phase

If the Organic Run is successful, but either the Recording phase or Replay phase fails, we recommend that you investigate the issue and then contact the TA infra team with a concrete problem statement. The TA infra team will help you fix the issue. For more information about when to contact the TA infra team, refer to this workflow.

![image1](https://user-images.githubusercontent.com/7036110/182635398-d137570d-da2f-4651-9284-ff293dbd6e1f.jpg)


### What happens if my test doesn’t pass during the Recording Phase?

If your mocked test doesn’t pass during the first run, your recording won't be saved until the test passes successfully. You can retry running your test as many times as you require.

### What if I need to update my test code after it's recorded?

If the changes that you are going to make to the current test are going to change, add or remove any API calls that have been recorded previously. You will need a new recording for that test. To get a new recording, increase the <code>testVersion<em> </em></code>in your <code>@MockedTest()<em> </em></code>annotation by at least one. For more information, refer to this example.

```kotlin
@MockedTest(testVersion = 2, mockServerVersion = 2)
```


If the code changes to your current test, don’t remove or introduce any API changes to the current test, then you don't need a new recording, and your test will just keep working normally.

### Would feature changes break our tests?


Suppose your feature has a recorded test on Mockingjay with, let's say, five API calls during the test. You just updated your feature by either adding a new API call, deleting an API call, or changing the sequence of the API call order for your feature. In this scenario, your test needs a new recording. But first, consider this question: 

**Are there any other tests in the project that is dependent on my feature?**

- If not, you can update the `testVersion` value_ _in the annotation and then re-record your test.
- If yes, changing the `testVersion` value_ _in the annotation won't be sufficient because it will only affect your test, not the other tests that depend on your feature.

Let's consider the log-in feature as an example, which is used in many other tests. Many features depend on the Login screen in their tests; for example, to test the User Profile screen, the test needs to go via the Login screen. If any API changes happen for the log-in feature, we must redo the recording for all other tests that depend on that feature.

In such a scenario, we need to increase the version number of that feature available in the `PageObject` of that screen or feature. This code shows an example of increasing the version number for the log-in feature. 

```kotlin
class LoginScreen @Inject constructor(
   private val buildInfo: BuildInfo
) : BasePageObject() {

   // ..........
   // ..........
  
   override fun getVersion(): Int {
       return 2
   }
}
```


After the version has changed, all tests that use that specific `PageObject` must be recorded in the next test run.

This is time-consuming because many tests need to be re-recorded, and some tests might be outdated and not work with staging again. Perform this action only if your feature change will affect other tests by making them fail. If the impact of the change will affect only your test, consider changing the `testVersion` value in the `@MockedTest` test annotation that will force the re-recording only for your test.


### Can I still use TaHeaderProvider.kt in Mockingjay?

Yes, ```TaHeaderProvider.kt``` supports Mock Server 1 and Mockingjay. However, there's one function inside this class that has no use case in Mockingjay and is deprecated now. We will remove it once we completely remove the implementation of Mock Server 1. This function is called ```setDefaultHeader()```

We don't want to support setting a default header for all paths in a test using ```TaHeaderProvider.kt``` simply because If such a scenario occurred, where we need to send a header for all the requests in a test, TA infra team will add the needed header in the Interceptor ```TaCustomHeaderInterceptor.kt``` which will be applied to all tests.

Other functions in this class are available to use. One useful use case, for example, is to pass a scenario ID for your API path to Mockingjay with the value of the ID of your scenario from the Mockingjay dashboard. This will return the desired response for the API path you added to the dashboard. Here's what it would look like.

```kotlin
    @MockedTest(testVersion = 1, mockServerVersion = 2)
    fun exampleTest() {
        taHeaderProvider.setDefaultHeaderForPath(
            "api/v1/feature-api-path",
            "ID of the response you added to the Mockingjay dashboard"
        )
    }
```

Now, during the test. Any API call made to `"api/v1/feature-api-path"` will get the response you specified in the dashboard


### Can I still use Mock-Server v1?

You can use Mock-Server v1 only if Mockingjay can’t cover your test case. For more information about Mock-Server v1, see [Mock Server](https://docs.google.com/document/d/1qQ2N9LOypljuMvfkwVcAhrrlyPZYpzzp40PhGrCaFD4/edit).

To use Mock-Server v1, change the mock server version in the annotation as shown here:  
**@MockedTest(testVersion = 1, mockServerVersion = 1)**

**Note**: If you find a test case that can't be covered with Mockingjay, we recommend that you contact the TA infra team. The team can help you fix the issue.

### Can I read more about the back-end implementation of Mockingjay?


For more information about how we implemented Mockingjay on the back end, see [Mockingjay](https://github.com/deliveryhero/mockingjay/wiki).

### How can I contact the TA infra team?

You can reach out to the TA infra team on these Slack channels:

- `#pd-mockingjay`

FAQ
