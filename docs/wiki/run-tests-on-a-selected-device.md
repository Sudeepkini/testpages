This wiki will explain how to run your tests on a device of your choice from Bitrise

Using the below environment variables. you can specify which device your tests will run against. If the environment variables are not specified, we will use the default values which you can see next to the variable below.

    $TEST_DEVICE_MODEL (default = Pixel2)
    $TEST_DEVICE_VERSION (default = 30)
    $TEST_DEVICE_LOCALE (default = en_US)
    $TEST_DEVICE_TIMEOUT (default = the argument --timeout which is passed when generating the flank config file)

This will allow you to run your tests on any device you want as long as it’s supported by FTL (Firebase Test Lab)  
If you want to get the list of available devices on FTL, you can execute this line  `gcloud firebase test android models list`

Or visit this  [link](http://json-parser.com/bb6930e6)  Which is a JSON file I fetched by executing the line aboveJust pass any of the environment variables listed above to your Bitrise build with valid values from the JSON file above and your tests will run on the specified device you have selected.

-   The value passed to  `$TEST_DEVICE_MODEL`  should be taken from this property value  `androidDeviceCatalog.models.id`  in the JSON file
-   The value passed to  `$TEST_DEVICE_VERSION`  should be one of the values of this object  `androidDeviceCatalog.models.supportedVersionIds`  in the JSON file
-   The value passed to  `$TEST_DEVICE_LOCALE`  should be one of the property values of  `runtimeConfiguration.locales.id`  from the JSON file
-   The value passed to  `$TEST_DEVICE_TIMEOUT`  is in minutes and should be a maximum of 45 if the selected device was a physical device and a maximum of 60 minutes if the selected device was a virtual device

Some device models don’t support all the API versions. So be careful of passing the wrong API version to a selected model.  
For example, the Pixel2 device (Which is the current default device for our tests) supports only the following version based on the JSON list.`"supportedVersionIds": [ "26", "27", "28", "29", "30" ]`As you can see, It doesn’t support API 31. So, if you pass this environment variable  `$TEST_DEVICE_VERSION=31`  to your Bitrise build. the build will fail because the default model we have is Pixel2 and you just overrode the version to be 31 which is not supported by FTL. if you want to run your tests on a device with API 31, make sure to also choose a device model that supports API level 31 from the JSON file and pass the model value to  `$TEST_DEVICE_MODEL`

**NOTE**: The JSON list contains both physical and virtual devices. Running your tests on a physical device should have a good reason (e.g trying to reproduce a specific crash or bug on a specific device). This is important because running tests on a physical device is more expensive than running them on a virtual device. Also, the availability of physical devices is limited and your build might need to wait for some time before a physical device becomes available.
