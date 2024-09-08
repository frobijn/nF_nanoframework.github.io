# Extending the framework

## It's the interfaces, not the attributes

The **nanoFramework** test platform loads a unit test assembly (after a build) to discover the tests. It then finds the test classes and test methods by investigating the class and method attributes. It recognises the attributes not by their name, but by the interface they implement:

| Interface | Function
| --- | --- |
| `ITestClass` | The class is a test class |
| `ISetup` | The method is a setup method |
| `ITestMethod` | The method of test class is a test method; also used to skip tests |
| `ICleanup` | The method is a cleanup method |
| `IDeploymentConfiguration` | The attribute provides the deployment configuration keys for the method's arguments |
| `IDataRow` | The attribute provides data for a test method's arguments |
| `ITraits` | The attribute specifies one or more traits for a test method, test class or test assembly |
| `ITestOnRealHardware` | The test method (or all test in the test class or test assembly) should be run on real hardware. The implementation of the interface checks whether a device satisfies the criteria. |
| `ITestOnVirtualDevice` | The test method (or all test in the test class or test assembly) should be run on a virtual device. |

For technical reasons assembly attributes cannot be applied to an assembly but instead should be applied to one or more classes that implements `IAssemblyAttributes`. The classes are also found by looking for the interface.

If you do not like the attributes that come out of the box, you can define your own. An attribute can implement multiple interfaces. E.g.,:

```csharp
namespace nanoFramework.TestFramework.MyExtensions
{
    [AttributeUsage(AttributeTargets.Method, AllowMultiple = false)]
    public class OutOfOrderAttribute : Attribute, ITestMethod, ITraits
    {
        #region ITestMethod implementation
        public bool CanBeRun
            => false;
        #endregion

        #region ITraits implementation
        public string[] Traits
            => new string[] { "Out of order" };
        #endregion
    }

    [AttributeUsage(AttributeTargets.Method, AllowMultiple = false)]
    public class TestOnDevBoardAttribute : Attribute, IDeploymentConfiguration, ITestOnRealHardware
    {
        #region IDeploymentConfiguration implementation
        public string[] ConfigurationKeys
            => "DevBoard configuration";
        #endregion

        #region ITestOnRealHardware implementation
        public string Description
            => "DevBoard"

        public bool ShouldTestOnDevice(ITestDevice testDevice)
        {
            byte[] configData = testDevice.GetDeploymentConfigurationFile (ConfigurationKeys);
            if (configData is null)
            {
                return false;
            }
            MyConfiguration configuration = MyConfiguration.Parse (configData);
            // Other criteria based on the content of the configuration
        }

        public bool AreDevicesEqual(ITestDevice testDevice1, ITestDevice testDevice2)
            => true;
        #endregion
    }
}

namespace nanoFramework.TestFramework.Test
{
    [TestClass]
    public class TestOfTest
    {
        [MyExtensions.OutOfOrder]
        public void TestThatIsNoLongerCorrect()
        {
        }

        [MyExtensions.TestOnDevBoard]
        public void TestHardware(byte[] configuration)
        {
        }
    }
}
```

Implement test platform extensions in a .NET **nanoFramework** class library that references the NuGet package `nanoFramework.TestFramework.Core`.

## Evaluation of the attributes

It is important to understand that you can use *any* **nanoFramework** code in the implementation of the attribute interfaces.

The test platform looks for the attributes while discovering the unit tests in a test assembly. The test assembly and all its dependencies are *not* loaded on a **nanoFramework** device but in a .NET Framework test host. All types in the framework's core library are mapped to corresponding types in *mscorlib*; other types should be present in one of the dependency assemblies.

If the test platform queries a test class or method for its attributes, the .NET Framework instantiates the attributes. That will fail if the attribute (indirectly) requires a type that cannot be mapped to *mscorlib* or one of the other libraries. If the test platform retrieves the value of an interface property or calls an interface method, the implementation should not use any code that is implemented in a native assembly, as those are not present. Also be aware of the subtle differences between .NET **nanoFramework** and .NET Framework, e.g., an enumeration converted to a string is a number in .NET **nanoFramework** and a name in .NET Framework.

The attributes are not used by the code that runs the unit tests on a **nanoFramework** device.

There is one exception: attributes that implement `IDataRow` are used in both the test host and on the **nanoFramework** device.

## Testing and debugging custom test attributes

For attributes that have straightforward interface implementations (e.g., `OutOfOrderAttribute` in the example), it is probably not worth the effort to create separate unit tests. That may be different for attributes that have non-trivial interface implementations (e.g., parsing of configuration data in `TestOnDevBoardAttribute`). 

To create a unit test for attributes that can be debugged, start by creating a class library that has all possible applications of the attributes you want to test: 

- Create a .NET **nanoFramework** class library that references the test framework extensions class library. The class library should *not* reference the `nanoFramework.TestFramework.UnitTestsProject` NuGet package.
- The attributes can be added to regular class (not necessarily test classes) and methods. To test assembly attributes, use classes that implement `IAssemblyAttribute`.

Then create the actual unit test project:

- Create a .NET Framework unit test project for .NET 4.8
- Reference the NuGet package `nanoFramework.TestFramework.Tooling`
- Make sure the .NET **nanoFramework** class library is built before the .NET Framework project. If the .NET Framework project references the .NET **nanoFramework** class library, Visual Studio generates a warning about the incompatibility of the projects but it will use the dependency to build the .NET **nanoFramework** class library first. There are also other ways to achieve that; see the Visual Studio documentation.

- Create unit tests that assert the proper functioning of the attributes in the .NET **nanoFramework** class library. In the unit tests:

    - Load the .NET **nanoFramework** class library assembly using `AssemblyLoader` in the `nanoFramework.TestFramework.Tooling` namespace.
    - All assembly attributes can be found using `AttributeProxy.GetAssemblyAttributeProxies` in the `nanoFramework.TestFramework.Tooling.TestFrameworkProxy` namespace. For each interface implemented by an attribute, the method returns a corresponding class derived from `AttributeProxy`.
    - All attributes of a class can be found by `AttributeProxy.GetClassAttributeProxies`. You cannot use `typeof()` to get the class, as the .NET Framework project cannot use the types in the .NET **nanoFramework** class library directly. Instead you have to enumerate all classes in the loaded assembly and find the class by its full name.
    - All attributes of a method can be found by `AttributeProxy.GetMethodAttributeProxies`. Get the method from its class's type using reflection (`GetMethod`).
    - Use the proxy classes to assert the proper functioning of the attributes.

Example:
```csharp
namespace nanoFramework.TestFramework.MyExtensions.Test
{
    [Test]
    public class TestOutOfOrderAttribute
    {
        [TestMethod]
        public void TestOutOfOrderForMethod ()
        {
            var testAssembly = AssemblyLoader.LoadFile ("..path to unit test assembly..");
            var testClass = (from t in testAssembly.GetTypes ()
                             where t.FullName == "nanoFramework.TestFramework.MyExtensions.OutOfOrderTestClass"
                             select t).First ();
            var allAttributes = AttributeProxy.GetMethodAttributeProxies (testClass.GetMethod ("OutOfOrderTestMethod"), new TestFrameworkImplementation (), null);

            var actualITraits = allAttributes.OfType (typeof (TraitsProxy)).First ();
            Assert.AreEqual ("Out of order", actualITraits.Traits[0]);

            var actualITestMethod = allAttributes.OfType (typeof (TestMethodProxy)).First ();
            Assert.AreEqual (false, actualITestMethod.CanBeRun);
        }
    }
}
```

Non-trivial implementations of `IDataRow` should also be tested in a regular .NET **nanoFramework** unit test projects.

## Monitoring the progress of unit tests on real hardware

It is common to run hardware-specific tests on a development board. Some boards have extra features, e.g., on-board LED or a small screen, that can be used to display some information about the tests being executed on that device. A visual cue that a device is taking parts in the tests can be useful, as the test platform cannot use the Visual Studio Test Explorer to communicate that information.

To implement such a monitor, create a class library with a public class that implements the `IUnitTestMonitor` interface. Mark the class with the appropriate `[TestOn...]` [attributes](writing-unit-tests#where-to-run-a-test-method-device-selection) to inform the test platform on what devices the monitor can be used. The monitor can have a setup method to receive deployment configuration data.

```csharp
namespace nanoFramework.TestFramework.MyExtensions
{
    [TestOnRealHardware]
    public class MonitorTestsUsingRGBLED : IUnitTestMonitor
    {
        [Setup]
        [DeploymentConfiguration("RGB LED I/O port")]
        public void Setup(string ioPort)
        {
        }

        #region IUnitTestMonitor implementation
        ...
        #endregion
    }
}
```

To avoid interference with the unit tests, the monitor is called before or after all tests of a test class are run and not between the execution of two test methods of the same test class.

Build and publish the monitor assembly and its dependencies to a directory, one directory per monitor. To use one or more monitors with unit tests, specify the paths to the directory in one of the `nano.runsettings`/`nano.runsettings.user` files:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RunSettings>
    <nanoFrameworkAdapter>

        <UnitTestMonitors>
            <Directory>Monitors\RGBLED</Directory>
            <Directory>Monitors\LCDScreen</Directory>
        </UnitTestMonitors>

        <!-- Other settings -->

    </nanoFrameworkAdapter>
</RunSettings>
```

If two .runsettings files specify unit test monitors, the list from the one last read replaces the list from the previous one.

The monitor can also be used with a [unit tests debug project](debugging-unit-tests). Use `DebugOnDevice.json` if the monitor should be included most of the time, and `SelectUnitTests.json` for one-time inclusion. If the file is specified in both, the one in `SelectUnitTests.json` is used:

```json
{
  "$schema": "obj/nF/SelectUnitTests.schema.json"

  "UnitTestMonitors": [
    "Monitors/RGBLED",
    "Monitors/LCDScreen"
  ]

  ... test cases ...
}
```

To test the monitor, create a .NET **nanoFramework** unit test project. Add a `nano.runsettings` with an empty `UnitTestMonitors` element. In the unit tests create an instance of the monitor, call its setup method (if it has one) and assert that it works properly.