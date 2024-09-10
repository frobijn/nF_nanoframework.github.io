# Deployment configuration

Especially in more generic hardware-specific tests extra information is required about the "make and model" of the device. It is not enough to know the platform or installed firmware. The [TestOn... attributes](writing-unit-tests#where-to-run-a-test-method-device-selection) may need to know whether the additional hardware (e.g., a sensor) is connected to a device in order to decide whether a test can be run on that device. When the test is executed, it needs to now which I/O ports the hardware is connected to.

The test platform has a mechanism to provide this type of *deployment configuration* to the test attributes and to test or setup methods that should run on real hardware. Other test frameworks have similar features (cf *TestRunParameters* in the VSTest test host). The test platform has no knowledge of what the content of the deployment configuration is - that is up to you. It only provides a mechanism to get that information to your code.

## Deployment configuration specification

The deployment configuration is a collection of key/value pairs. The key/value collection is specified in a json file with a file name you choose. Its content is:

```json
{
    "DisplayName": "Development kit #1",
    "Configuration": 
    {
        "RGB LED I/O port": 48,
        "SSID name": "DevRouter",
        "DevBoard configuration": { "File": "../DevBoards/TestBoard_S3N32R8V_OV2640.cfg" }
    }
}
```

The key identifies what the significance is of the value. There are no special keys, you decide how to name the keys.

Values can be specified either as a textual or integer value in the json file, or as the path to a file that contains the data. Files can be specified using a path relative to the directory the json file resides in, and forward slashes can be used in the path. Your code can access the content of a file either as a textual (`string`), integer (`int` or `long`) or binary (`byte[]`) data. If the value is specified in the deployment configuration file, your code can access the value only as textual or integer value.

## Specifying deployment configuration for unit tests

The .NET **nanoFramework** has no mechanism to retrieve deployment configuration from the actual device via the serial port. Instead you have to specify what deployment configuration should be used for a device connected to a specific COM port. You typically specify that in a `nano.runsettings.user` [configuration file](controlling-the-test-execution), e.g.:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RunSettings>
    <nanoFrameworkAdapter>

        <DeploymentConfiguration>
            <SerialPort>COM9</SerialPort>
            <File>MyBoard_v3.json</File>
        </DeploymentConfiguration>

        <DeploymentConfiguration>
            <SerialPort>COM11</SerialPort>
            <File>MCU_only_S3N32R8V.json</File>
        </DeploymentConfiguration>

        <!-- Other settings -->

    </nanoFrameworkAdapter>
</RunSettings>
```

The `SerialPort` specifies the port to use for communication with the device. The presence of a deployment configuration is not enough to make the device available for unit tests; if `AllowRealHardware` is present it has to be `true`, and if `AllowSerialPorts` is present it should list the serial port.

`File` specifies the path to the json file with the deployment configuration. The path can be relative to the directory the .runsettings file resides in.

The deployment configuration specification follows the same hierarchy rules as other .runsettings: if a .runsetting file that is read later specifies a new file path for a specific serial port, that file path is used by the test platform. The test platform does not merge the deployment configuration specifications from both files, it uses the file that is specified as latest.

## Specifying deployment configuration for a debug unit tests project

To specify the deployment information to use in a [unit tests debug project](debugging-unit-tests), provide the name to the deployment configuration file in one of the json files. Use `DebugOnDevice.json` if the configuration does not change often, and `SelectUnitTests.json` if it does. If the file is specified in both, the one in `SelectUnitTests.json` is used:

```json
{
  "$schema": "obj/nF/SelectUnitTests.schema.json"

  "DeploymentConfiguration": "MyBoard_v3.json"

  ... test cases ...
}
```

## Using deployment configuration information

The deployment configuration can be passed to [setup and test methods](writing-unit-tests) using the `[DeploymentConfiguration]` attribute:

```csharp
namespace nanoFramework.TestFramework.Test
{
    [TestClass]
    public class TestOfTest
    {
        [Setup]
        [DeploymentConfiguration ("DevBoard configuration", "SSID name")]
        public void TestHardware(byte[] configuration, string ssidName)
        {
        }

        [TestMethod]
        [DeploymentConfiguration ("RGB LED I/O port")]
        public void TestRGBLED (int ioPort)
        {
        }

        [DataRow(3, 1)]
        [DataRow(4, 1)]
        [DataRow(5, 9)]
        [DeploymentConfiguration ("Test image", "Device ID")]
        public void TestRGBLED (byte[] image, long deviceId, int datarow_1, int datarow_2)
        {
        }
    }
}
```
The `[DeploymentConfiguration]` attribute lists the keys in the deployment configuration. The method should have corresponding arguments of type `string` to receive textual values, `int` or `long` for integer values or `byte[]` for binary data. If data is not available, the argument is `null` or -1 for integer values; this is reported in the result of the unit test.

The deployment configuration can be used to decide whether a test can be executed on a (real hardware) device. The test platform does not provide any attributes out of the box for this purpose, but you can easily provide one yourself. Your attribute can be used instead of the `[TestOn...]` [attributes](writing-unit-tests#where-to-run-a-test-method-device-selection) for real hardware:

```csharp
namespace nanoFramework.TestFramework.MyExtensions
{
    [AttributeUsage(AttributeTargets.Method, AllowMultiple = false)]
    public class TestOnDevBoardAttribute : Attribute, ITestOnRealHardware
    {
        public string Description
            => "DevBoard"

        public bool ShouldTestOnDevice(ITestDevice testDevice)
        {
            byte[] configData = testDevice.GetDeploymentConfigurationFile ("DevBoard configuration");
            if (configData is null)
            {
                return false;
            }
            MyConfiguration configuration = MyConfiguration.Parse (configData);
            // Other criteria based on the content of the configuration
        }

        public bool AreDevicesEqual(ITestDevice testDevice1, ITestDevice testDevice2)
            => true;
    }
}

namespace nanoFramework.TestFramework.Test
{
    [TestClass]
    public class TestOfTest
    {
        [MyExtensions.TestOnDevBoard]
        public void TestHardware()
        {
        }

        [Setup]
        [DeploymentConfiguration ("DevBoard configuration")]
        public void TestHardware(byte[] configuration)
        {
        }
    }
}
```
As the code of the attribute is executed [on the testhost](extending-the-framework#evaluation-of-the-attributes) and not on a **nanoFramework** device, the `ShouldTestOnDevice` and `AreDevicesEqual` cannot use the specialized framework libraries. If you need those, perform the check in the setup method:

```csharp
namespace nanoFramework.TestFramework.Test
{
    [TestClass]
    public class TestOfTest
    {
        [TestOnRealHardware]
        public void TestHardware()
        {
        }

        [Setup]
        [DeploymentConfiguration ("DevBoard configuration")]
        public void TestHardware(byte[] configuration)
        {
            if (/* cannot run for this configuration */)
            {
                Assert.SkipTest();
            }
        }
    }
}
```
It is recommended to use a custom attribute for as much of the deployment configuration check as possible, as it is executed before any tests are executed on a device. If the outcome is that none of the selected tests have to be run on the device, the test platform can skip deploying the test assembly to the device. The evaluation of a setup method is always done on the device, so after deployment to the device.