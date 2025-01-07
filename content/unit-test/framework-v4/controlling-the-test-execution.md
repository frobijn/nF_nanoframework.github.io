# Controlling the test execution

The author of a unit test has indicated via [attributes](writing-unit-tests#where-to-run-a-test-method-device-selection) on what devices the test should be run. There are two configuration files that control how the tests should be run:

- Tests are run on the nanoDevices that satisfy the criteria expressed in the [`nano.devices.json` configuration](../versioning/nano-devices-json.md) for the test project. If that file does not exist or if no device selection is configured, the test platform makes a selection from all available (connected) nanoDevices. Tests are run on [as many devices as required](run-tests-in-visual-studio#running-the-unit-tests); it may not be necessary to run tests on all available devices.

- How the tests should be run can be specified via an optional [`nano.tests.json` test configuration](#test-configuration-file). The test configuration is needed if the tests require additional data via the [deployment configuration](deployment-configuration.md).

## Test configuration file

The test configuration is specified in the optional `nano.tests.json` file in the test project's directory:

```json
{
    "RealHardwareTimeout": 120000,
    "MaxVirtualDevices": 4,
    "VirtualDeviceTimeout": 60000,
    "Logging": "None",
    "DeploymentConfiguration": [
        "../deployment.json"
    ]
}
```

with:

- `RealHardwareTimeout` is the maximum time in milliseconds the execution of the tests in a single test assembly on real hardware is allowed to take. This is excluding the time it takes to initialize the device and deploy the tests to the device.
- `MaxVirtualDevices` is the maximum number of virtual devices to run in parallel. Specify 0 to use as many as the computer has logical processors.
- `VirtualDeviceTimeout` is the maximum time in milliseconds the execution of the tests in a single test assembly on the virtual device is allowed to take.
- `Logging` specifies the logging of the test platform during the test discovery and execution orchestration. The logging can be viewed in the output window of Visual Studio for test discovery, and in the test results for the execution orchestration. Valid values are `None`, `Detailed`, `Verbose`, `Warning` and `Error`. If omitted `Warning` is used.
- `DeploymentConfiguration`: see [deployment configuration](deployment-configuration).

## Configuration of the VSTest test host

For technical reasons the configuration of the tests for the .NET **nanoFramework** test platform is different from the usual configuration (a .runsettings file) employed by Visual Studio. That is: if you dig deep enough you'll find that the test platform still uses a .runsettings file to instruct Visual Studio and VSTest how to run tests. The test platform creates a `nano.runsettings` file in the same directory the test assembly ends up, with only the settings required by the test platform:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RunSettings>
    <RunConfiguration>
        <MaxCpuCount>1</MaxCpuCount>
        <TargetFrameworkVersion>net48</TargetFrameworkVersion>
        <TargetPlatform>x64</TargetPlatform>
        <TestAdaptersPaths><!-- location of the nanoFramework test platform adapter --></TestAdaptersPaths>
    </RunConfiguration>
</RunSettings>
```

The main purpose of this file is to make sure Visual Studio can find the test adapter for discovering and running unit tests, without any action required by the developer. Do not change this file; it will be re-generated on the next build.



