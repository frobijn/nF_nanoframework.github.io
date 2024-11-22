# Writing unit tests

The basic premise of writing unit tests is that the author of the test knows exactly what the test is for, how it should be run and on what devices it makes sense to execute the test. That is communicated to the test platform tooling via attributes. This section describes the standard attributes and their effect.

## Test classes and methods
To create a unit test, add a class to the project and add a method for each test.
The classes and tests methods are recognized using attributes. You'll find the following ones:

- `[TestClass]`: this attribute is used on a class, without this attribute on the calls, the class won't be considered as a a valid Unit Test and the test methods inside will be ignored. You can have as many classes as you want with this attribute.
- `[TestMethod]`: this attribute is used on any method where you'll have tests running inside. You can have as many methods with this attribute as you want. The `TestMethod` attribute can be omitted if any of the other test platform attributes are present.
- `[DataRow(...)]`: this attribute is used on test methods with arguments. Every `[DataRow(...)]` attribute specifies a set of parameters to run the test method with. You can have as many attributes per test method as you want.

Test classes and methods must be `public`, otherwise they will not be discovered. Test methods should have no (`void`) return type.

Here is a typical example of how you can use the attributes:

```csharp
namespace nanoFramework.TestFramework.Test
{
    [TestClass]
    public class TestOfTest
    {
        [TestMethod]
        public void TestStringComparison()
        {
            // Perform tests for string comparison
        }

        [DataRow(1,2)]
        [DataRow(42,43)]
        public void TestAddByOne(int actual, int expected)
        {
            // Perform tests for the test data and verify it has the expected value
        }
    }
}
```

As you can see in this example, you just use the attributes to decorate the class and the functions.

For more information on how verify the test results, see [Assert test results](assert).

## Setup/cleanup of a test context

Often all test methods in a test class share a test context that have to be created before the test proper can be executed. The test context may also have to be cleaned up once the test is completed. Initialization of the test context can be done in a default constructor of the test class and/or setup methods. The test context can be cleanup via cleanup methods and/or an implementation of the `IDisposable` interface:

```csharp
namespace nanoFramework.TestFramework.Test
{
    [TestClass]
    public class TestOfTest : IDisposable
    {
        public TestOfTest()
        {
            // The constructor will be called first (if present)
        }

        [Setup]
        public void RunSetup()
        {
            // A setup method will be called next (if present)
        }

        [TestMethod]
        public void Test()
        {
            // Then the test method(s) are called
        }

        [Cleanup]
        public void RunCleanup()
        {
            // The cleanup method is called after the test method(s)
        }

        public void Dispose ()
        {
            // Finally IDisposable.Dispose is called (if implemented)
        }
    }
}
```

- It is technically possible to have both a constructor/`IDisposable` pair and setup/cleanup methods. However, it is recommended that you choose either the constructor/`IDisposable` method or create setup/cleanup methods. Setup/cleanup methods are required if access to the [deployment configuration](deployment-configuration) is required for the initialization of the test context.

- `[Setup]`: this attribute is used on any method. This method will be called first. While you technically have as many of those functions per class, it is recommended to only use 1 per class. Typical usage is to setup hardware you'll need to have running for all the tests methods.
- `[Cleanup]`: this attribute is used on any method. This method will be called last, after the all the tests methods. While you technically have as many of those functions per class, it is recommended to only use 1 per class.

**Important**: if the initialization of the test context fails, none of the test methods for which the initialization is performed will be executed. For this reason it is recommended that a setup/cleanup or constructor/`IDisposable` is used to initialize, e.g., a device or other hardware required by the tests, or to verify whether the device that runs the test has the required features. If the constructor/setup methods succeed, all tests fill be executed and, regardless of the outcome of the tests, the cleanup/Dispose methods will be called.

## When to run setup/cleanup methods

In many situations it is sufficient for the setup and cleanup methods to run before and after all tests in a test class. E.g., if the setup method initialises the I/O ports of the MCU for the hardware under test. There are also cases where that is not sufficient, e.g., if the configuration of I/O ports is modified as part of the test. In order for each test method to have the same starting point, irrespective of whether previous tests succeeded or failed, the setup/cleanup methods have to be run before and after each test method.

The test platform supports that via a parameter to the `TestClass` attribute:
```csharp
namespace nanoFramework.TestFramework.Test
{
    [TestClass (true /* = run setup/cleanup for every test method */)]
    public class TestOfTest
    {
        [Setup]
        public void RunSetup()
        {
            // The setup method will be called before Test1 and Test2
        }

        [TestMethod]
        public void Test1()
        {
        }


        [TestMethod]
        public void Test2()
        {
        }

        [Cleanup]
        public void RunCleanup()
        {
            // The cleanup method is called after the Test1 and Test2
        }
    }
}
```

## When to instantiate a test class

In most unit test frameworks test classes are non-static classes that are instantiated before any setup or test methods are called. As the .NET **nanoFramework** assemblies run on hardware with limited capabilities, instantiating classes comes at a cost that is waisted if the setup/cleanup and test methods do not use non-static fields. That is why it is also allowed to create static test classes. A static test class works the same as a non-static class, except it is not instantiated:

```csharp
namespace nanoFramework.TestFramework.Test
{
    [TestClass (... /* = same as for non-static classes */)]
    public static class TestOfTest
    {
        [Setup]
        public static void RunSetup()
        {
        }

        [TestMethod]
        public static void Test()
        {
        }

        [Cleanup]
        public static void RunCleanup()
        {
        }
    }
}
```

For non-static classes the same considerations apply as for the setup/cleanup methods: sometimes it is sufficient to instantiate a test class once for all test methods, sometimes a new instance of a test class should be created for each test method. In order to limit the number of instances of test classes, the default is one instance for all test methods. A developer can change that behaviour via a second argument to the `TestClass` attribute;

```csharp
namespace nanoFramework.TestFramework.Test
{
    [TestClass (true, true /* = one instance per test method */)]
    public class TestOfTest
    {
        public TestOfTest()
        {
            // Called before Test1 and before Test2
        }

        [TestMethod]
        public void Test1()
        {
        }

        [TestMethod]
        public void Test2()
        {
        }
    }
}
```

## Where to run a test method: device selection

The test platform supports running unit tests on real hardware devices and on a virtual device. The virtual device runs on the Windows platform and provides support for the non-hardware-related features in the .NET **nanoFramework** platform. In the future the support of other features (e.g., networking) may be included.

The support of the test platform for testing on real hardware and on a virtual device is different. The test platform can run the tests from multiple test assemblies in parallel using multiple virtual devices, whereas multiple test assemblies have to be run one by one on a hardware device. Every virtual device has the same capabilities so it doesn't matter on which virtual device is executed, but there is a wide variety of hardware devices and a test may have to be run on several different models. A virtual device can always be started, but real hardware is not always available.

The test platform requires the author of a test to separately indicate whether a test should be run on a virtual device, and whether a test should be run on real hardware. The author can also indicate what type of real hardware device. Example:

```csharp
namespace nanoFramework.TestFramework.Test
{
    [TestClass]
    public class TestOfTest
    {

        [TestOnVirtualDevice]
        public void TestOfDataProcessing()
        {
            // This method is only executed on the virtual device
        }

        [TestOnRealHardware]
        [TestOnVirtualDevice]
        public void TestGpioExtensions()
        {
            // This method is executed on real hardware and on the virtual device
        }

        [TestOnPlatform ("esp32")]
        public void TestEsp32SpecificCLRImplementations()
        {
            // This method is only executed on an ESP32 device
        }
    }
}
```

Of course the tests that should be run on real hardware are only executed if that hardware is available. The test platform selects the devices to run (a selection of) tests on:

- All selected tests that should be run on a virtual device, are executed on a virtual device. Each test assembly is run on a separate virtual device. If needed multiple virtual devices are run in parallel.

- Tests on real hardware are run one test assembly at a time, one assembly after the other. The test platform tries to run tests on as many real hardware devices as available, in parallel (and in parallel with any virtual devices).

- For each available real hardware device, the test platform checks whether a test should be run on that device. If two devices meet the criteria for the same test, the test is run on at least one of the devices. The test is also run on the other device if the author of the test has indicated that the two devices are sufficiently different that it makes sense to execute the test on both devices.

The attributes that determine what device is selected:

- The `[TestOnVirtualDevice]` attribute indicates that the test should be executed on the virtual device.

- The `[TestOnRealHardware]` attribute indicates that the test should be executed on real hardware. It is sufficient to run the test on one of the available real hardware devices.

- The `[TestOnPlatform]` attribute indicates that the test should be executed on real hardware of a particular platform. If two available devices have different firmware (a different target), the test should be run on both devices.

- The `[TestOnTarget]` attribute indicates that the test should be executed on real hardware that has the specified firmware (target) installed. The test should be run on only one of the available devices.

- If you want to have different selection criteria for a real hardware device, create a new attribute that implements the `ITestOnRealHardware` interface.

If the author of a test does not specify on what devices a test should be executed, the test platform acts as if the `[TestOnVirtualDevice]` and `[TestOnRealHardware]` attributes have been specified.

## Assembly and test class attributes

The selection of devices to run a test on may be identical for all tests in a class. The corresponding attributes may also be set for the test class:

```csharp
namespace nanoFramework.TestFramework.Test
{
    [TestClass]
    [TestOnVirtualDevice]
    public class TestOfTest
    {
        public void TestOfDataProcessing()
        {
            // This method should be executed on the virtual device
        }

        [TestOnPlatform ("esp32")]
        [TestOnPlatform ("stm32")]
        public void TestUsingDeviceDependentCLRFeature()
        {
            // This method should be executed on an ESP32 device, a STM32 device and on the virtual device.
        }
    }
}
```

It is not uncommon that all tests in a test assembly are designed to run on the same type of devices. The attributes may also be applied to the assembly as a whole. For technical reasons it is not possible to use the attributes as assembly attributes. Instead the author of the test assembly should add a public class that implements the `IAssemblyAttributes` interface and apply the attributes to that class:

```csharp
namespace nanoFramework.TestFramework.Test
{
    [TestOnVirtualDevice] // All tests in the assembly are run on the virtual device
    public class AssemblyAttributes : IAssemblyAttributes
    {
    }
}
```

You can add as many classes that implement `IAssemblyAttributes` as you like.

## Other attributes

- Test can be categorized by using the `[Trait]` attribute. This is discussed in [Run tests in Visual Studio](run-tests-in-visual-studio#custom-traits). The `[Trait]` attribute can be used with a test method, test class or test assembly.

- A setup or test method can receive selected deployment configuration data via the `[DeploymentConfiguration]` attribute; see [Use a deployment configuration](deployment-configuration).

- Everyone can add new attributes that extend the behaviour of one or more out-of-the-box attributes; see [Extend the framework](extending-the-framework).