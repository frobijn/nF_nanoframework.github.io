# Where tests are executed

## Selection of the nanoDevice

By default the test platform selects the nanoDevice based on the characteristics of the test project's assembly and the assemblies it depends on. Each test will be run on at most one nanoDevice.

If the test assembly can be deployed to a Virtual nanoDevice because it does not depend on any hardware features, the test platform will start a Virtual nanoDevice for each test project to execute the tests. If necessary, multiple Virtual nanoDevice are started in parallel up to a [configurable maximum](controlling-the-test-execution.md#test-configuration-file).

If a test assembly cannot be deployed to a Virtual nanoDevice, the test platform checks the (real hardware) nanoDevices connected to the computer running Visual Studio or VSTest. If the test assembly can be deployed to one of the connected nanoDevices, the test platform will do so. If there are multiple connected nanoDevices and the tests to run are from multiple test projects, the test platform tries to run the tests from various projects in parallel on different nanoDevices (and in parallel with the tests executing on the Virtual nanoDevices).

Some or all tests in a test project can make use of [deployment configuration](deployment-configuration.md), i.e., "make and model" information like the GPIO ports peripherals are connected to, or the settings of a WiFi access point. If tests require that data and cannot be executed without the data, the test platform will also take the availability of the deployment configuration into account. It will try to find a nanoDevice that an run all (selected) tests. If that is not possible, it will split the tests from one project into separate groups and execute the maximum amount of groups on different nanoDevices.

If a test assembly cannot be executed to any of the connected devices, the test platform will show that in the result of the tests. The tests will be marked as skipped rather than as failed.

## Controlling where to run the test

The default nanoDevice selection of the test platform is based on the premise that only your (.NET) code has to be tested and not the .NET **nanoFramework** firmware/runtime or its interaction with the microcontroller. If your code works on one nanoDevice, it will also work on a nanoDevice with different microcontroller or different firmware/runtime.

That may not be true. Your code may depend on hardware-specific features that behave differently on different platforms. E.g., if your code requires high-precision math, you may want to test the code not only on the Virtual nanoDevice but also on real hardware nanoDevices that cannot handle double precision  calculations natively. If you are using several versions of custom firmware/runtimes, you may also want to test your .NET code on all firmware versions.

If you write unit tests for which the default device selection is not sufficient, you can control where the tests are executed by using one or more of the attributes `TestOnVirtualDevice`, `TestOnEachTarget`, `TestOnRealHardware`, `TestOnPlatform` and `TestOnTarget`:

```csharp
[TestClass]
public class MyTestClass
{

    [TestOnVirtualDevice]
    public void TestOfDataProcessing()
    {
        // This method is only executed on the virtual device
    }

    [TestOnRealHardware]
    [TestOnVirtualDevice]
    public void TestMethod()
    {
        // This method is executed on both the virtual device and a single hardware nanoDevice
    }

    [TestOnRealHardware(true)]
    public void TestHighPrecisionMath()
    {
        // This method is executed only on hardware nanoDevices
        // If connected devices have different firmware/runtime, it is executed on a single
        // device for each of the available runtimes.
    }

    [TestOnPlatform ("esp32")]
    public void TestEsp32SpecificCLRImplementations()
    {
        // This method is only executed on one device with firmware from the ESP32-platform.
    }
}
```

The attributes can also be applied to the test class:

```csharp
[TestClass]
// All test method inherit the attribute:
[TestOnVirtualDevice]
public class MyTestClass
{
    public void TestOfDataProcessing()
    {
        // This method is only executed on the virtual device
    }

    [TestOnRealHardware]
    public void TestMethod()
    {
        // This method is executed on both the virtual device and a single hardware nanoDevice
    }
}
```

Or to a class that implements the `ITestAssembly` interface:

```csharp
// Execute all tests in the assembly on as much nanoDevices with different
// firmware/runtimes, including the Virtual Device.
[TestOnEachTarget]
public class AssemblyAttributes : ITestAssembly
{
}
```

If a test method has no `TestOn...` attributes and does not inherit one from its test class or from a class that implements the `ITestAssembly` interface, the default behaviour applies: the test will be executed on a single nanoDevice, and the test platform determines which one. The test will be executed only once, even if there is another test in the test project that has attributes instructing it to be executed on multiple devices.

If one or more `TestOn...` attributes apply to a test method, the test platform will select the nanoDevices based on the attributes. If that requires executing the test on multiple nanoDevices, that is what the test platform will do. The test platform may choose to represent the same test with multiple test cases in the Visual Studio Test Explorer, to provide you with the opportunity to [select where to run the test](run-tests-in-visual-studio.md).

## The `nano.devices.json` configuration

If your project under test uses a [`nano.devices.json` configuration](../../versioning) to ensure the software can be deployed to a selection of firmware/targets of a particular version, create a `nano.devices.json` configuration for the test project as well.

The test platform honours the [device selection](../../versioning/nano-devices-json.md) in the `nano.devices.json` via the `Platforms`, `DeviceTypes` and `DeviceSelection` and `FirmwareArchivePath` specifications. Only nanoDevices that satisfy the selection criteria will be taken into account. Within that selection the test platform will choose the best devices to execute the tests on.

As a result, the device selection for the test project may be different from the one for the software under test. Example: the software under test may have a `nano.devices.json` configuration:

```json
{
    "Import": [ "../Global/nano.devices.json" ],
    "Platforms": "esp32"
}
```

but if the test project can be executed on the Virtual Device, its `nano.devices.json` configuration should look like:

```json
{
    "Import": [ "../Global/nano.devices.json" ],
    "DeviceTypes": "Virtual nanoDevice"
}
```

If your project is using a firmware archive, make sure (the correct version of) the runtime of the Virtual nanoDevice is present in the firmware archive.

The presence of a `nano.devices.json` configuration also has effect on the presentation of the tests in the Visual Studio Test Explorer. Example: a test method has attribute `TestOnEachTarget` and in `nano.devices.json` a selection of devices is specified:

```json
{
    "FirmwareArchivePath": "../Firmware",
    "Platforms": "esp32"
}
```

The test platform will create a test case for each target in the firmware archive for the ESP32-platform. In the Visual Studio Test Explorer that looks like:

TODO: image of Visual Studio Test Explorer.

Provided you've connected a nanoDevice with each target to the computer, you can select in the Test Explorer on which 
firmware/target to run the test and you can keep track on which targets a test has already been executed.
