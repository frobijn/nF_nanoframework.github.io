# Deployment configuration

Especially in more generic hardware-specific tests extra information is required about the "make and model" of the device. It is not enough to know the platform or installed firmware. The [TestOn... attributes](writing-unit-tests#where-to-run-a-test-method-device-selection) may need to know whether the additional hardware (e.g., a sensor) is connected to a device in order to decide whether a test can be run on that device. When the test is executed, it needs to now which I/O ports the hardware is connected to.

The test platform has a mechanism to provide this type of *deployment configuration* to the test attributes and to test or setup methods that should run on real hardware. Other test frameworks have similar features (cf *TestRunParameters* in the VSTest test host). The test platform has no knowledge of what the content of the deployment configuration is - that is up to you. It only provides a mechanism to get that information to your code.

## Structure of the deployment configuration

The purpose of the deployment configuration is to specify settings as *key* = *value* pairs for a nanoDevice that can run tests. As the same settings may apply to multiple nanoDevices, the configuration consists of *components* that are groups of settings. The structure of the deployment configuration is:

- Deployment configuration to be used for a test project.

    The configuration consists of:

    - Component (zero or more) designed for a (class of) nanoDevice.

      The nanoDevice is identified by exactly one of:
        - Platform the nanoDevice is part of, e.g., ESP32, STM32, ti_simplelink, gg11 (case insensitive).
        - Name of the firmware/target installed on the nanoDevice.
        - The system serial number of the device.
        - The module serial number of the device.

      The settings for the nanoDevice are specified as:
        - Name of the component (zero or more) to include the settings of. The settings are included in the order in which the components are listed. If multiple components specify a *value* for the same *key*, the last value is used.
        - *key* = *value* setting (zero or more); this overwrites the value for the same *key* from the included components.

The deployment configuration for a specific device is obtained by combining the components that match the device's platform, firmware/target name, system serial number and module serial number (in that order). If multiple values are specified for the same *key*, the last value encountered is used.

## Specification of the deployment configuration

The deployment configuration is specified in one or more files. A deployment configuration file an also import other files; those files are read first and the the content of this file is used to overwrite the imported settings. A file is read only the first time it is encountered, if it is encountered a second time it is not read again.

Each deployment configuration file is a json file (comments allowed) with a file name you choose. Its content is:

```json
{
    "Import": [
        "devices.deployment.json"
    ],

    "Components": 
    {
        "SSID": {
            "Platform": "ESP32",
            "Configuration": {
                "SSID name": "DevRouter"
            }
        },

        "Thread": {
            "Target": "ESP32_C6_THREAD",
            "Configuration": {
                "PAN ID": "0xBEEF",
                "XPAN ID": "0xBEEF1111CAFE2222"
            }
        },

        "ESP32-S3-DevKitC": {
            "Configuration": {
                "RGB LED I/O port": 48,
            }
        },

        "My primary test device": {
            "SSN": "100000000068B6B33CDA80",
            "Use": [
                "ESP32-S3-DevKitC",
                "SSID"
            ],
            "Configuration": {
                "DevBoard configuration": { "File": "../DevBoards/TestBoard_S3N32R8V_OV2640.cfg" }
            }
        }
    }
}
```

with:

- `Import` (optional) is a an array with one or more paths to other deployment configuration files. The files are imported in the order they are specified.
- `Components` is an json object that specify the components. The object's property names are the names of the components. Each component is a json object with properties:
    - A selector of a nanoDevice (optional), one of: 
        - `Platform`: platform the nanoDevice, e.g., ESP32, STM32, ti_simplelink, gg11 (case insensitive).
        - `Target`: the name of the firmware/target of the nanoDevice. Use `Virtual nanoDevice` to indicate the deployment data is for a virtual device.
        - `SSN`: the system serial number of the nanoDevice.
        - `MSN`: the module serial number of the nanoDevice.

        The system and module serial number can be obtained by requesting the *Device capabilities* for a connected nanoDevice in the device explorer in Visual Studio. The .NET **nanoFramework** firmware provides a default system serial number for several platforms.

    - `Use` is an array of component names to include the settings of. The settings are included in the order the components are listed. If multiple components contain a value for the same *key*, the last value encountered is used.

    - `Configuration` is a json object. Each property is a setting, with the property name as *key*. The value of the property can be one of:
        - A string.
        - An integer number.
        - An object with a single property `File` and as value the path to the file that contains the *value*.

A path to a file can be specified relative to the directory the configuration file resides in. It can also be an absolute path, and the path may contain environment variables like `%USERPROFILE%`. Instead of a `\` a '/' may be used. So `../.nanoFramework/deployment.json`, `c:\ProgramData\nanoFramework\deployment.json` and `%USERPROFILE%/.nanoFramework/deployment.jsondeployment.json` are all valid paths.

## Specifying deployment configuration for unit tests

To use deployment configuration data for a test project, specify the deployment configuration file(s) in the [`nano.tests.json` configuration file](controlling-the-test-execution#test-configuration-file), e.g.:

```json
{
    "DeploymentConfiguration": [
        "deployment.json"
    ]
}
```

The `DeploymentConfiguration` is an array one or more paths to other deployment configuration files, similar to *Import* in a deployment configuration file. The files are imported in the order they are specified. A path to a file can be specified relative to the test project directory. It can also be an absolute path, and the path may contain environment variables like `%USERPROFILE%`. Instead of a `\` a '/' may be used. So `../global/devices.deployment.json`, `c:\ProgramData\nanoFramework\devices.deployment.json` and `%USERPROFILE%/.nanoFramework/devices.deployment.json` are all valid paths.

## Specifying deployment configuration for a debug unit tests project

The test platform derives the deployment configuration required for a debug unit tests project from the configuration specified for the test projects. If one or more values in the resulting deployment configuration depend on the (class of) nanoDevice the debug project is deployed to, one of the deployment options should be selected in `SelectUnitTests.json`:

```json
{
  "$schema": "obj/nF/SelectUnitTests.schema.json"

  "DeployTo": {
    "Target": "ESP32_S3_ALL"
  }

  ... test cases ...
}
```

The `DeployTo` object has a single property, one of `Platform`, `Target`, `SSN` or `MSN`. Which ones are allowed and what values can be chosen is part of the JSON schema `SelectUnitTests.schema.json` and is available via intellisense in the Visual Studio editor.

## Using deployment configuration information

The deployment configuration can be passed to [setup and test methods](writing-unit-tests) of of test classes and classes that implement `ITestAssembly` using the `[DeploymentConfiguration]` attribute applied to the parameter that should receive the configuration value:

```csharp
[TestClass]
public class MyTestClass
{
    [Setup]
    public void TestHardware(
        [DeploymentConfiguration ("DevBoard configuration")] byte[] configuration,
        [DeploymentConfiguration ("SSID name")] string ssidName)
    {
    }

    [TestMethod]
    public void TestRGBLED ([DeploymentConfiguration ("RGB LED I/O port")] int ioPort)
    {
    }

    [DataRow(3, 1)]
    [DataRow(4, 1)]
    [DataRow(5, 9)]
    public void TestRGBLED (
        DeploymentConfiguration ("Test image")] byte[] image,
        int datarow_1,
        int datarow_2,
        DeploymentConfiguration ("Device ID")] long deviceId)
    {
    }
}
```
The `[DeploymentConfiguration]` attribute accepts the key in the deployment configuration. The parameter should have a type of `string` to receive textual values, `int` or `long` for integer values or `byte[]` for binary data. If data is not available, the argument passed is `null` or -1 for integer values; this is reported in the result of the unit test.

It is best to specify the value in the deployment configuration with the same type as used in the code: a text value or file for `string` data, a number for `int` or `long` and a file for `byte[]`. The test platform will try to convert values from one type to another if necessary. A value that is used in the code as an `int` can be specified in the deployment configuration as a string, eg., `"42"`.

## Executing tests depending on deployment configuration information

By default the test platform will not execute tests if not all required deployment configuration is available. E.g., if a setup method of a test class requires one value (eg., with key "SSID name") from the deployment configuration and a test method requires another (e.g, "url"), no test of the test class is executed if the deployment configuration does not provide a value for "SSID name", and the test method is executed only if values for "SSID name" and "url" are present.

You can tell the test platform that it is acceptable to execute a method if a value is not present by using `false` as the second argument of the `DeploymentConfiguration` attribute:

```csharp
[TestClass]
public class MyTestClass
{
    [Setup]
    public void TestHardware(
        [DeploymentConfiguration ("SSID name", false)] string ssidName)
    {
    }

    [TestMethod]
    public void TestWebsiteAccess ([DeploymentConfiguration ("url", false)] string ioPort)
    {
    }
}
```

If no value is available for a particular *key*, the default value `null` (or -1 for integer values) is passed to the method.

The test platform does not provide any attributes out of the box if it is more complicated to determine when a setup or test method can be run given a particular deployment configuration. But you can easily provide one yourself. Define an attribute that implements the `ITestOnConfiguredDevice` interface and apply it to any setup or test method:

```csharp
[TestClass]
public class MyTestClass
{
    [TestOnDevBoard]
    [Setup]
    public void Setup()
    {
    }

    [TestMethod]
    public void TestHardware([DeploymentConfiguration ("DevBoard configuration")] byte[] configuration)
    {
    }
}

[AttributeUsage(AttributeTargets.Method, AllowMultiple = false)]
public class TestOnDevBoardAttribute : Attribute, ITestOnConfiguredDevice
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
```
As the code of the attribute is executed [on the testhost](extending-the-framework#evaluation-of-the-attributes) and not on a **nanoFramework** device, the `ShouldTestOnDevice` and `AreDevicesEqual` cannot use the specialized .NET **nanoFramework** libraries. If you need those, perform the check in the setup method:

```csharp
[TestClass]
public class MyTestClass
{
    [TestOnDevBoard]
    [Setup]
    public void Setup()
    {
        byte[] configData = testDevice.GetDeploymentConfigurationFile ("DevBoard configuration");
        if (configData is null)
        {
            return false;
        }
        MyConfiguration configuration = MyConfiguration.Parse (configData);

        // Other criteria based on the content of the configuration

        if (/* cannot run for this configuration */)
        {
            Assert.SetupFailed("The test cannot be executed on this device.");
        }
    }

    [TestMethod]
    public void TestHardware([DeploymentConfiguration ("DevBoard configuration")] byte[] configuration)
    {
    }
}
```

It is recommended to use a attributes (rather than code in a setup or test method) for as much of the deployment configuration check as possible, as attributes are evaluated before any tests are executed on a device. If the outcome is that none of the selected tests have to be run on the device, the test platform can skip deploying the test assembly to the device. The evaluation of a setup method is always done on the device, so after deployment to the device.
