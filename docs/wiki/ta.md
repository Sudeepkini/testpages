## Workshop
For those who are new to UI Automation, please refer to Android TA Workshop as a guide.

[Android TA Workshop](https://github.com/deliveryhero/pd-android-ui-testing-workshop)

Also, using TA template will be useful in writing tests faster. Please refer to this guide for setting up custom TA template

[Custom TA Template](https://github.com/deliveryhero/pd-android-ui-testing-workshop/wiki/How-to-use-Android-TA-Templates)

## TestRail Annotations
![171560512-cf430ab7-69c0-49cd-97f0-d219c4b6e84b](https://user-images.githubusercontent.com/16208059/171561713-3f72dee1-cd99-4a5d-8563-79fd3ddf648d.png)

We have two types on TestRail, including Core Test and Regression. The Priority is just to help developers to prioritize which test to work on first and has no relation to annotations.

### 1. Core Test
Core Test means `@CriticalTest`.

Please make sure all critical tests run on Mock Server, as critical tests will be run when raising a new PR, which could block other's PR.


### 2. Regression
Regression means `@RegressionTest`, run during the regression testing and every other night on Tuesday, Thursday and Saturday at 20:00.


## Tribe annotation
![截圖 2022-06-02 下午1 38 15](https://user-images.githubusercontent.com/16208059/171562568-c1295748-31f5-486d-991e-717239d5b8c7.png)

We have above tribe annotations mapping to tribes in [Pandora Quality Dashboard](https://datastudio.google.com/u/0/reporting/8e2b5a06-5191-4032-9431-ffdce24bf50a/page/GElTC). Please specify your tribe annotations when adding new TAs.

## Review TA reports without VPN
If VPN is not available, you can try this guide to [review TA reports without VPN](https://confluence.deliveryhero.com/display/GLOBAL/Review+TA+Reports+without+VPN)
