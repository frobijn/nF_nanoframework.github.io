# Controlling the test execution

The author of a unit test has [indicated](where-tests-are-executed.md) on what devices the test should be run. There are two configuration files that control how the tests should be run:

- Tests are run on the nanoDevices that satisfy the criteria expressed in the [`nano.devices.json` configuration](../versioning/nano-devices-json.md) for the test project. If that file does not exist or if no device selection is configured, the test platform makes a selection from all available (connected) nanoDevices. Tests are run on [as many devices as required](run-tests-in-visual-studio#running-the-unit-tests); it may not be necessary to run tests on all available devices.

- How the tests should be run can be specified via an optional [`nano.tests.json` test configuration](#test-configuration-file). The test configuration is needed if the tests require additional data via the [deployment configuration](deployment-configuration.md).

## Test configuration file

The test configuration is specified in the optional `nano.tests.json` file in the test project's directory:

```json
{
    "$schema": "https://raw.githubusercontent.com/nanoframework/nanoFramework.TestFramework/refs/heads/main/schemas/nano.tests.json",
    "Logging": "None",
    "MaxTestHosts": "4",
    "DeploymentConfiguration": [
        "../deployment.json"
    ],
    "MaxVirtualDevices": 4,
    "VirtualDeviceTimeout": 60000,
    "RealHardwareTimeout": 120000
}
```

with:

- `Logging` (optional) specifies the logging of the test platform during the test discovery and execution orchestration. The logging can be viewed in the output window of Visual Studio for test discovery, and (if generated as part of the execution of tests) in the test results. Valid values are `None`, `Detailed`, `Verbose`, `Warning` and `Error`. If omitted `Warning` is used.
- `MaxTestHosts` (optional) is the maximum number of test hosts to run in parallel. When the test platform is hosted in Visual Studio or VSTest, it will spin up one or more test hosts to discover unit tests and orchestrate the execution of tests on the available platforms. If supported by Visual Studio/VSTest, multiple test hosts run in parallel. The default is to use as many as the computer has logical processors for discovery, and as many as needed for the orchestration of the test execution.
- `DeploymentConfiguration` (optional) specifies the deployment configuration files in the same way as the `Import` element of the [deployment configuration](deployment-configuration.md).
- `MaxVirtualDevices` (optional) is the maximum number of virtual devices to run in parallel. Defaults to *MaxTestHosts*.
- `VirtualDeviceTimeout` (optional) is the maximum time in milliseconds the execution of the tests in a single test assembly on the virtual device is allowed to take.
- `RealHardwareTimeout` (optional) is the maximum time in milliseconds the execution of the tests in a single test assembly on a real hardware nanoDevice is allowed to take. This is excluding the time it takes to initialize the device and deploy the tests to the device.

The *MaxTestHosts* and *MaxVirtualDevices* are not relevant for a single test project, as the test platform will only use a single test host and at most one virtual device for the project. If Visual Studio/VSTest asks the test platform to do work for multiple test projects, it will read the configurations of all projects and use the minimum of the *MaxTestHosts* and *MaxVirtualDevices* settings.
 
## Configuration of the VSTest test host

For technical reasons the configuration of the tests for the .NET **nanoFramework** test platform is different from the usual configuration (a .runsettings file) employed by Visual Studio. That is: if you dig deep enough you'll find that the test platform still uses a .runsettings file to instruct Visual Studio and VSTest how to run tests. The test platform creates a `nano.vs.runsettings` file in the same directory the test assembly ends up, with only the settings required by the test platform:

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

It still is possible to use a custom configuration file to instruct Visual Studio and VSTest how to run tests for configuration that is not related to the .NET **nanoFramework** test platform. Create a file `nano.runsettings` in the project directory and add all configuration options. At build time the content of the file will be copied to the `nano.vs.runsettings` file (which is located in a different directory than `nano.runsettings`), and:

- In the `RunConfiguration` section:
    - The `MaxCpuCount`, `TargetFrameworkVersion`, `TargetPlatform` and `TestAdaptersPaths` settings will be added or overwritten.
    - The `DotnetHostPath` and `TestCaseFilter` elements will be removed.
- The `TestRunParameters` and `nanoFrameworkAdapter` sections are removed.

