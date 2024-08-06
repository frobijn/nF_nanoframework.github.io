# Run tests in Visual Studio

## Viewing and selecting tests

The tests from a .NET **nanoFramework** test project are visible in the Visual Studio Test Explorer, where also tests show up from the same solution created with other test frameworks. The default view is by project and class name:

TODO: image

There are other ways to organize the tests; see the *Group By* button in the Test Explorer. Via the *Columns* option in the Test Explorer's *Options* menu you can show extra information for the tests. For .NET **nanoFramework** test assemblies, all information except *Stack Trace* and *Target Framework* is supported.

Tests that should be run both on the virtual device and on real hardware are shown twice in the Test Explorer, once for each type of device. This allows you to control the type of device a test should be executed on, by selecting the corresponding test. Unfortunately the Test Explorer has no mechanism to show only the tests that can be run on the available real hardware that actually is connected to your computer, or to automatically hide test that should be run on real hardware if no devices are available. However, you can hide tests for a device type via the filter at the top right corner of the Test Explorer:

- `Trait:"@Virtual nanoDevice"` shows only tests that should be executed on the Virtual Device and hides test for real hardware and for regular .NET target frameworks;
- `-Trait:"@Virtual Device"` hides tests that should be executed on the Virtual Device;
- `Trait:"@Hardware nanoDevice"` shows only tests that should be executed on real hardware and hides test for the virtual device and for regular .NET target frameworks;
- `-Trait:"@Hardware nanoDevice"` hides tests that should be executed on real hardware;

The Test Explorer has a memory for recent filters, and has a button to remove the filter.

## Custom traits
If you are using traits to organize and select the tests: you can add custom traits to test methods:

```csharp
namespace nanoFramework.TestFramework.Test
{
    [Trait("Trait for all tests in the test project")]
    public class AssemblyAttributes : IAssemblyAttributes
    {
    }

    [TestClass]
    [Trait("Trait for all tests in the test class")]
    public class TestOfTest
    {
        [Trait("Trait for this test method")]
        public void Test()
        {
            // This method should be executed on the virtual device
        }
    }
}
```

The Test Explorer shows three extra traits for the test method in the example.

## Running the unit tests

In the Test Explorer you select the tests you want to run. Visual Studio then instructs the various test frameworks to run the tests. .NET **nanoFramework** test platform fully supports this. If you select a single test, only that test is executed. If you select all tests, all are executed.

The test platform uses the [TestOn... attributes](writing-unit-tests#where-to-run-a-test-method-device-selection) to decide on which device a test should be executed.

For tests that should be run on the virtual device, the test platform starts a single virtual device per unit test projects to run the tests from that project. If multiple virtual devices are required, they are run in parallel. The virtual devices are stopped after the selected tests have been run.

For tests that should be run on real hardware, the test platform examines which of the available devices can run the selected tests, based on the test attributes. The selected tests are deployed and run on the device one test project at a time. The test platform tries to optimize the device selection to shorten the time it takes to run the unit tests. Tests on different devices run in parallel.

**Beware** that the preparation of a real hardware device to run tests can require a lot of time (tens of seconds). The total time it takes is reported in the test results. Errors are also reported in the test results. Change the [logging](controlling-the-test-execution#content-of-a-configuration-file) to *Verbose* to include information about the steps within the initialization, or to *Detailed* to get even more information. With *Detailed* logging the test result also shows why a test was (not) run on an available real hardware device.

Which of the available devices are actually used can be limited via a [per-user configuration](controlling-the-test-execution#configuration-file-hierarchy) or by temporarily disconnecting a device. If you have reserved serial ports for the Virtual nanoDevice, make sure to add the ports to *ExcludeSerialPorts* (or exclude the ports from *AllowSerialPorts*).

## Viewing test results

TODO: image of test result for virtual device test

Once a test has been run, the outcome of the test is shown in the Test Explorer. There are four possible outcomes:

- A test has been skipped: there was no suitable (real hardware) device to run the test on, or after the test was started, the test signalled that it should not be run on the device.
- A test passed
- A test failed. Either the test proper failed, or the test context could not be initialized via the constructor/setup methods, or the cleanup via the cleanup/dispose methods failed. If it is not the test proper that failed, the error message will indicate that. (The tests can be filtered on the error message.)

The details of the test show the usual information. A link to the source code of the test is available for all test methods defined in the test project. The *Environment* shows which device the test was executed on; this can be used to filter the test results. The output of the test is divided in several sections:

- The output of the test method
- The output of the constructor/setup method. If the constructor/setup method is shared by multiple tests, the output is shown in the details of each test.
- The output of the cleanup/dispose method. If the cleanup/dispose method is shared by multiple tests, the output is shown in the details of each test.
- The information about the deployment of the test assembly to the device

If a test has been executed on multiple real hardware devices, the details for each device are shown:

TODO: image of test results with multiple devices.

## Debugging unit tests

The Test Explorer not only lets you run a unit test, but also start a selection of tests in the debugger. This is not supported by the .NET **nanoFramework** test platform. An error message is displayed in the *Tests* section of the Visual Studio *Output* window, and no tests will be started.

The test platform has an alternative that offers exactly the same functionality but in a different way: a [unit tests debug project](debugging-unit-tests).

## Running unit tests simultaneously from multiple instances of Visual Studio

It is not recommended to run unit tests simultaneously from multiple instances of Visual Studio and/or [VSTest](run-tests-in-vstest)).
The test platform does not fail but has no means of optimizing the execution of the tests. The total time it takes to execute the tests may increase significantly.