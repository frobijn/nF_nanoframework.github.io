# .NET **nanoFramework** Unit Tests platform v2

**nanoFramework** has long offered a complete Unit Tests platform called `nanoFramework.TestFramework`. Version 2 of the platform offers an improved user experience, fine-grained control over the execution of tests and an easy way to debug unit tests.

## What is nanoFramework.TestFramework

nanoFramework.TestFramework is a Unit Test platform dedicated to .NET **nanoFramework**! It has all the benefits of what you're used to when using Microsoft Test platform for .NET or XUnit or any other!

Working with the nanoFramework.TestFramework is almost as easy as working with any other platform:

1. [Create a unit test project](Create-a-unit-test-project)

	The Visual Studio extension for the .NET **nanoFramework** provides a special template to create a Unit Test project. All required components are added to the project via a single NuGet package. If an update to the framework of Visual Studio integration becomes available, all you have to do is to upgrade the NuGet package.

2. [Write unit tests](writing-unit-tests)

	It is easy to get started; you'll have the first unit test running in minutes! As with other platforms, the nanoFramework.TestFramework provides several options to code the tests. There is extensive support to assert that the result ot the unit tests are valid. And if tests are executed on a virtual device the nanoFramework.TestFramework has several optimization features to run tests in parallel, similar to many modern platforms.

	An aspect that is unique to nanoFramework.TestFramework is the ability to run tests on real hardware! The framework gives the author of a a unit test several tools to indicate where the test is intended to be executed. Some tests do not need to be run on real hardware as they only use the non-hardware related parts of the .NET **nanoFramework**, for example tests for data processing tasks. Other tests can be executed only on real hardware as they require features not present in a virtual device, like I/O pins, SPI or I2C support and connected sensors or other devices. And then there are tests that must be executed on specific hardware as it uses features that are unique to a particular device. Once the author has specified the intended platform for a test, the nanoFramework.TestFramework tools ensure that the test will only be executed on those platforms.

3. [Assert test results](assert)

	The test platform provides extensive support to verify the results of the test. It is also possible to signal whether a test fails because the test context is not properly initialized or disposed of, or whether a test cannot be run because the device does not support some required features.

4. [Run tests in Visual Studio](run-tests-in-visual-studio)

	The nanoFramework.TestFramework is well integrated with the Test Explorer in Visual Studio. You can view, select and execute tests as with any other platform. All the usual criteria for the selection of tests are available, like project, class, state and traits. But you can also select tests based on the platform they are intended to be executed on, making it easy to switch from running tests on a virtual device to executing the same or other tests on hardware connected to your computer. As you would expect, the state of the tests are visible in the Test Explorer, including an indication of the reason: whether a test failed because it could not be started, or whether it failed or succeeded.

5. [Debug unit tests](debugging-unit-tests)

	Because of the unique devices .NET **nanoFramework** runs on, be it real hardware or a virtual device, is it not possible to debug unit tests from the Test Explorer. But the nanoFramework.TestFramework has an alternative that works just as well: a special type of project for debugging unit tests.

	The Visual Studio extension for the .NET **nanoFramework** provides a special template to create a Unit Test Debugger project. You only have to reference the unit test project and specify which test or tests should be debugged. Then you can use the same powerful debugger that is available to debug any other .NET **nanoFramework** application.

6. [Controlling the test execution](controlling-the-test-execution)

	Sometimes you need to impose further limitations on the test execution that are specific to the way tests are run. On a build/test server it is in general not possible to run any or all tests on a hardware device. You can add all test projects to a special solution and then limit execution of the tests to, e.g., the virtual device only. The nanoFramework.TestFramework employs the same mechanism as the Visual Studio test engines, a .runsettings file, but has custom rules how to organize the files. By keeping the nanoFramework.TestFramework separate from the regular configuration files it is possible to combine tests for .NET **nanoFramework** and for regular .NET platforms within the same solution and repository.

7. [Use a deployment configuration](deployment-configuration)

	...

8. [Extend the framework](extending-the-framework)

	Out of the box the nanoFramework.TestFramework has almost everything you'll need. Almost, because your project may have unique requirements that are not covered by the framework. You're in luck: the framework allows you to replace and extend the framework's attributes for annotating tests. And if you require extra tooling to work with the tests: the core of the Visual Studio integration and unit test debugging tools is available as NuGet package, to base your own tools on.

9. [Migrate from nanoFramework.TestFramework v2](migrate-from-v1)

	You may already be using the v2 version of the nanoFramework.TestFramework and perhaps you already have a lot of test projects. Can you easily migrate to v3 of the framework? Yes indeed! The first step is as easy as updating the nanoFramework.TestFramework to the latest version, and you'll be able to use most improvements right away. Read the detailed description for the next few steps to modify the projects to enjoy all benefits of version 3.

10. [Run tests in VSTest](run-tests-in-vstest)

	The test platform fully supports running tests in an automated build/test environment via VSTest.

11. [Use custom tooling](use-custom-tooling)

	If you want to create a custom tool that needs to analyse a .NET **nanoFramework** test assembly, you can use the test platform code to do the heavy lifting.

Because of the unique nature of the .NET **nanoFramework** there are still some [constraints and limitations](constraints-limitations). 

If you are interested into the architecture, please check out [this detailed page](../architecture/unit-test.md). The description is for v1 of the framework, but the architecture is essentially unchanged in v2.