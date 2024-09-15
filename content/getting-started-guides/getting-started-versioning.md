# Versioning guide

Once you have created and run a *Hello World* application, you have learned that the .NET **nanoFramework** consists of several packages and tools:

- Your application uses NuGet packages to import a selection of the framework's .NET class libraries. In general there's one class library per namespace.
- You have to install one of the suitable firmware packages on your device.
- There are tools like the [Visual Studio](getting-started-managed.md) or [VSCode](getting-started-vs-code.md) extensions, [nano Firmware Flasher (nanoff)](https://github.com/nanoframework/nanoFirmwareFlasher) and the [Virtual Device](virtual-device.md) that help you develop and deploy your application.

All packages and tools have version numbers, and at first sight most of the packages do not have an explicit dependency on other packages. You may be wondering whether you are free to mix and match packages and tools. And as you may have feared: the answer is no. Each NuGet package requires a specific version of the firmware or of the Virtual Device and some packages are not supported by a particular firmware package or cannot be run in the Virtual Device. If the framework supports a new type of device, the *nanoff* tool may have to be updated. If you are interested in a more detailed description, see the [application deployment](../architecture/deployment.md) description in the architecture section.

How can you make sure that your application uses matching versions of all packages and tools? The .NET **nanoFramework** has extensive support for two versioning strategies:

- [Auto-update](#auto-update): always use the latest version. If one of the framework tools signals a version discrepancy, simply update to the latest version. This is the easiest way to use the framework.
- [Controlled update](#controlled-update): select a single version of all package and tools to develop and test your application. Plan to update regularly to the latest version, but update the packages and tools at a time that suites you. Use this strategy if you require more control over the update process.

By default the .NET **nanoFramework** is configured for the auto-update strategy. You have to [prepare](#prepare-for-controlled-updates) your projects, solution and/or repository if you want to select the controlled update strategy.

## Auto-update

The CI/CD pipeline of the .NET **nanoFramework** ensures that all packages and tools are consistent with each other at any moment in time. All packages and tools are updated daily (around 0:00 UTC) with the latest enhancements and bug fixes, and if packages depend on each other, the references are updated. In principle, that is. Some tools and packages hardly ever change, there are many days without updates. But if there are enhancements and bug fixes, they will be made available as quickly as possible. If you use the framework long enough, there will be updates that affect your work.

Updates of the NuGet packages are notified via the package manager in Visual Studio. If there are new versions of the packages, always update to the latest version. Some updates require an update of the firmware of your device. That will be automatically detected when you deploy your application (via F5 in Visual Studio) to the device: an error message tells you that there is a version mismatch. Just update the firmware to the latest version.

If a new version of the firmware becomes available, the NuGet packages will also be updated. If you always use the latest NuGet packages and update the firmware if that is indicated in the F5-deployment, you don't have to monitor separately for updates of the firmware.

Every .NET **nanoFramework** tool auto-update itself, or at least shows a message if there is a new version available.

## Controlled update

Especially in commercial projects the auto-update strategy may not be feasible. An update of one of the packages or firmware may require planning and coordination with other teams. All your development and testing is done with a single version of the framework's packages and tools. You should still plan to update to the latest version of the framework regularly, if only to profit from bug fixes. But the actual update to a new version (if there is one) is planned and executed when there is time to assert that your application and device is still working correctly after the update.

Instead of using the latest version, you are using a version that is frozen in time. At the start of a project, you make a local copy of the firmware packages for your devices and of the *nanoff* tool and Virtual Device. You only use the NuGet packages that are current at that time. If you deploy firmware to a (new) device, you'll use the local copy of *nanoff* and of the firmware. In unit testing and debugging you use the local copy of the Virtual Device.

While you are developing, it may become necessary to use additional NuGet packages. Or you may encounter an issue with one of the NuGet packages that can be resolved by a bug fix of the class library only. (The community welcomes any PR with a resolution of an issue!) If your project allows to start using new (versions of) NuGet packages, you want to be sure from the start that the packages are consistent with the frozen version of the firmware. Add the *nanoFramework.Versioning* NuGet package to each project, it will verify the consistency of NuGet packages and the selected firmware versions when the software is built.

If you are ready to update to a new version, freeze the then-current version of the NuGet and firmware packages and tools. Deploy the new firmware to test devices and re-test your application. If all tests succeed, keep the version and proceed to develop/test with that version.

It is not necessary to use local copies of the Visual Studio and VSCode extensions. It is not expected that there will be future changes in the deployment of applications to a device that require an update of the firmware or NuGet packages.

## Prepare for controlled updates

To use the controlled update strategy, you have to configure your projects, solutions and/or repository. To demonstrate how that is done, the description in this section applies to the simplest case where you have a repository with projects and solutions for an application that runs on device type(s) that require one or more firmware packages. More complex cases can be configured in similar ways.

Assuming you are using a git/version-controlled repository to store the local copies, create a directory at the root of the repository, e.g., `<repository>\nanoFramework`. Install local copies of the .NET **nanoFramework** tools:

```
dotnet tool install nanoff --tool-path <repository>\nanoFramework\Tools
dotnet tool install nanoclr --tool-path <repository>\nanoFramework\Tools
```

Collect the firmware packages you need for your devices. E.g.:
```
<repository>\nanoFramework\Tools\nanoff --target ESP32_S3_ALL --updatefwarchive --fwarchivepath <repository>\nanoFramework\Firmware
```
If you don't know what firmware to use but have the device connected to the PC, use *nanoff --preview* to find the most suitable firmware.
If you are not sure which devices you will be using, collect all packages for a platform.
```
<repository>\nanoFramework\Tools\nanoff --platform ESP32 --updatefwarchive --fwarchivepath <repository>\nanoFramework\Firmware
```
From now on, if you want to ready a device for use in the projects, use the extra *fwarchive*-options to use the local copy of the firmware packages rather than the ones from the online repository, e.g.:
```
<repository>\nanoFramework\Tools\nanoff --suppressnanoffversioncheck --listtargets --serialport COM3 --update --fromfwarchive --fwarchivepath <repository>\nanoFramework\Firmware
<repository>\nanoFramework\Tools\nanoff --suppressnanoffversioncheck --platform esp32 --serialport COM3 --update --fromfwarchive --fwarchivepath <repository>\nanoFramework\Firmware
```
The *--suppressnanoffversioncheck* is optional and stops *nanoff* from checking whether a new version of *nanoff* is available. Even if the option is omitted and there is a new version, *nanoff* will not automatically be updated.

Create a configuration file `<repository>\nanoFramework\nano.devices.json` with the path to the local copy of the Virtual Device and a list of the device types you plan to use. You provide a name for the device type (that is not *Virtual nanoDevice*) and specify the firmware to use, e.g.:
```json
{
    "PathToLocalNanoCLR": "Tools/nanoclr.exe",
    "VirtualDeviceSerialPort": "COM30",
    "FirmwareArchivePath": "Firmware",
    "DeviceTypes": {
        "Primary device": "ESP32_S3_ALL",
        "Alternative": "ESP32_S3_BLE"
    }
}
```
If you plan to use the [test framework](../unit-test/framework-v3/index.md), create the `<repository>\nanoFramework\nano.runsettings` file:
```xml
<?xml version=""1.0"" encoding=""utf-8""?>
<RunSettings>
    <nanoFrameworkAdapter>
        <PathToLocalNanoCLR>Tools\nanoclr.exe</PathToLocalNanoCLR>
    </nanoFrameworkAdapter>
</RunSettings>
```
Optionally set other [global options](../unit-test/framework-v3/controlling-the-test-execution.md) via this file.

For each .NET **nanoFramework** project add the *nanoFramework.Versioning* NuGet package. The package installs an MSBuild task that verifies after each build whether the project's assembly and its dependencies can be deployed to the intended device types. The task gets its data from a `nano.devices.json` file in the project directory that lists the devices the project will be deployed on, either for testing, debugging or as part of the final product: 
```json
{
    "GlobalSettingsDirectoryPath": "relative path to <repository>\nanoFramework",
    "DeviceTypes": {
        "Primary device",
        "Virtual nanoDevice"
    }
}
```
Instead of device names the platform can also be used, e.g.:
```json
{
    "GlobalSettingsDirectoryPath": "relative path to <repository>\nanoFramework",
    "DeviceTypes": {
        "Virtual nanoDevice"
    },
    Platforms: {
        "ESP32"
    }
}
```
The `nano.devices.json` file is used only by the MSBuild task; it is not used by the test platform to determine on which of the connected devices the tests should be executed, and the Visual Studio/VSCode extension allows for the deployment of an application to any connected device.

If the project is a test project, also add a `nano.runsettings` file:
```xml
<?xml version=""1.0"" encoding=""utf-8""?>
<RunSettings>
    <nanoFrameworkAdapter>
        <GlobalSettingsDirectoryPath>...relative path to <repository>\nanoFramework...</GlobalSettingsDirectoryPath>
    </nanoFrameworkAdapter>
</RunSettings>
```

Optionally create a `nano.devices.json` file in a folder that contains a Visual Studio solution that contains projects that can be deployed via F5 to a Virtual Device with only:
```json
{
    "GlobalSettingsDirectoryPath": "relative path to <repository>\nanoFramework"
}
```
The Visual Studio extension will read this and use the local copy of the virtual device and serial port as defaults when starting the Virtual Device in the Device Explorer.
