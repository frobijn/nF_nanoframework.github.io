# .NET **nanoFramework** Unit Tests platform v4

**nanoFramework** has long offered a complete Unit Tests platform called `nanoFramework.TestFramework`. Version 3 of the platform offers an improved user experience, fine-grained control over the execution of tests and an easy way to debug unit tests.

## What is nanoFramework.TestFramework

nanoFramework.TestFramework is a Unit Test platform dedicated to .NET **nanoFramework**! It has all the benefits of what you're used to when using Microsoft Test platform for .NET or XUnit or any other!

Working with the nanoFramework.TestFramework is almost as easy as working with any other platform:

1. [Create a unit test project](Create-a-unit-test-project.md)

    The Visual Studio extension for the .NET **nanoFramework** provides a special template to create a Unit Test project. All required components are added to the project via a single NuGet package. If an update to the framework of Visual Studio integration becomes available, all you have to do is to upgrade the NuGet package.

2. [Write unit tests](writing-unit-tests.md)

    It is easy to get started; you'll have the first unit test running in minutes! The nanoFramework.TestFramework provides several options to code the tests, as explained in this section.

3. [Assert test results](assert.md)

    The test platform provides extensive support to verify the results of the test. It is also possible to signal whether a test fails because the test context is not properly initialized or disposed of, or whether a test cannot be run because the device does not support some required features.

4. [Where tests are executed](where-tests-are-executed.md)

    An aspect that is unique to nanoFramework.TestFramework is the ability to run tests on real hardware! If a test does not make use of hardware specific features, there is even a faster way to execute the test: using the Virtual nanoDevice that emulates a real device. You don't have to do anything, the test platform knowns when it can use a Virtual nanoDevice. It can also determine which the suitability of the nanoDevices that are connected to the computer running Visual Studio (or VSTest) to run the tests.

    If the test is designed for hardware related features that are implemented differently for the various types of nanoDevices (different firmware), the test should be executed on multiple devices. Or the test may be relevant only for a selection of the nanoDevices that are capable to run the test. There are several ways to instruct the test platform on what type of devices should be executed.

5. [Run tests in Visual Studio](run-tests-in-visual-studio.md)

    The nanoFramework.TestFramework is well integrated with the Test Explorer in Visual Studio. You can view, select and execute tests as with any other platform. All the usual criteria for the selection of tests are available, like project, class, state and traits. But you can also select tests based on the type of devices they will be executed on, making it easy to switch from running tests on a virtual device to executing the same or other tests on hardware connected to your computer. As you would expect, the state of the tests are visible in the Test Explorer, including an indication of the reason: whether a test failed because it could not be started, or whether it failed or succeeded.

6. [Debug unit tests](debugging-unit-tests.md)

    Because of the unique devices .NET **nanoFramework** runs on, be it real hardware or a virtual device, is it not possible to debug unit tests from the Test Explorer. But the nanoFramework.TestFramework has an alternative that works just as well: a special type of project for debugging unit tests.

    The Visual Studio extension for the .NET **nanoFramework** provides a special template to create a Unit Test Debugger project. You only have to reference the unit test project and specify which test or tests should be debugged. Then you can use the same powerful debugger that is available to debug any other .NET **nanoFramework** application.

7. [Controlling the test execution](controlling-the-test-execution.md)    
    The nanoFramework.TestFramework has several optimization features to select the nanoDevices to execute tests on and to shorten the time it takes to run all tests. Tests are run on the Virtual Device if possible, and are executed in parallel if multiple (hardware) nanoDevices are available.

    The test platform offers several options to tweak the test execution. E.g., to limit the tests to be run on the same type of nanoDevices that the software under test is intended to be deployed to.

8. [Use a deployment configuration](deployment-configuration.md)

    Your tests may depend on features that depend on the hardware related features other than the microcontroller. Like which I/O pins a peripheral is connected to, the GPIO port the on-board LED is wired to, which WiFi access point to use. If this "make and model" or deployment information is coded into the tests, the test would fail if it is run on a device that is wired differently. Or if a different computer in a different location is used to start the tests.

    The test platform has a solution: the deployment configuration. You can specify the "make and model" settings for each nanoDevice and the test platform will select the data for the device a tests is executed on. Or provide configurations that are specific for the computer that starts running the tests. The test platform will take the presence of deployment configuration into account when selecting the devices to run the tests on.

    The same mechanism is available for the projects that debug unit tests.

9. [Extend the framework](extending-the-framework.md)

    Out of the box the nanoFramework.TestFramework has almost everything you'll need. Almost, because your project may have unique requirements that are not covered by the framework. You're in luck: the framework allows you to replace and extend the framework's attributes for annotating tests. And if you require extra tooling to work with the tests: the core of the Visual Studio integration and unit test debugging tools is available as NuGet package, to base your own tools on.

10. [Migrate from nanoFramework.TestFramework v3](migrate-from-v1.md)

    You may already be using the v2 or v3 version of the nanoFramework.TestFramework and perhaps you already have a lot of test projects. Can you easily migrate to v4 of the framework? Yes indeed! The first step is as easy as updating the nanoFramework.TestFramework to the latest version, and you'll be able to use most improvements right away. Read the detailed description for the next few steps to modify the projects to enjoy all benefits of version 4.

11. [Run tests in VSTest](run-tests-in-vstest.md)

    The test platform fully supports running tests in an automated build/test environment via VSTest.

12. [Use custom tooling](use-custom-tooling.md)

    If you want to create a custom tool that needs to analyse a .NET **nanoFramework** test assembly, you can use the test platform code to do the heavy lifting.

Because of the unique nature of the .NET **nanoFramework** there are still some [constraints and limitations](constraints-limitations). 

If you are interested into the architecture, please check out [this detailed page](../architecture/unit-test.md). The description is for v1 of the framework, but the architecture is essentially unchanged in v3.
