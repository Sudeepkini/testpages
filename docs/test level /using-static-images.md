For preventing the size of apk increasing due to addition of static png for different dimension we always prefer to use vector images but In case there is only PNG available(Due to vector paths complexity) then those static images should be uploaded to S3 Bucket through [Images Server](https://images.deliveryhero.net/)

1- You need to get access of the Images Server through helpdesk from [Link](https://jira.deliveryhero.com/plugins/servlet/desk/portal/65/create/971) with below parameter filled up as in below image
[Image](https://images.deliveryhero.io/image/-test-/Screenshot%202021-01-13%20at%201.16.51%20AM.png)

2- Approval of your Manager will be required on above request and then you will get access by the [#dh-iam](https://deliveryhero.slack.com/archives/C7A9XTPJ4)

3- You can login now to the portal and ask for your team member for specific folders/url where your squad uploads images for e.g [Link](https://images.deliveryhero.net/images/fd-bd/) goes for pickup squad.

4- Now you uploaded your images in specified folder/sub-folders of your squad and then get the url from that.

5- You have to add a new Entry for the config type in S3 repo provided in [Link](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/app-configuration). Refer to the pull request [Here](https://github.com/deliveryhero/static.fd-api.com/pull/693) in which entry for each country config json files has been added. Merging this to master will be required before going to next one.

6- Now you need to have changes in modal(Pojo) class for the Static Config which are in [Repo](https://github.com/deliveryhero/pd-metalcore-android/). Refer to the [PR](https://github.com/deliveryhero/pd-metalcore-android/commit/b76434702512e75c0c31bab602bed2b06d0912dc) in relation to the 5th Point PR.

7- Now after (6th Point) PR is merged you can release a new version for Metalcore Library and include that in main consumer app PR(B2C Project) where you can use the newly added config and image from there through ImageLoader library.
