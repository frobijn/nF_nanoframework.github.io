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

To use the controlled update strategy, you have to configure your projects, solutions and/or repository. To demonstrate how that is done, the description in this section applies to the simplest case where you have a repository with projects and solutions for an application that runs on device type(s) that require one or more firmware packages.

### Use local copies of tools

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
If you expect the framework version not to be updated for a long time (e.g. in case your product has a long lifetime and software development is paused for most of the time), also keep a separate instance of the runtime of the *nanoclr.exe* tool, *WIN_DLL_nanoCLR*:
```
<repository>\nanoFramework\Tools\nanoff --target WIN_DLL_nanoCLR --updatefwarchive --fwarchivepath <repository>\nanoFramework\Firmware
```
If in future the *nanoclr.exe* tool has to be updated, e.g., because of dependencies on third-party software, this Virtual Device runtime that matches your application is still available.

From now on, if you want to ready a device for use in the projects, use the extra *fwarchive*-options to use the local copy of the firmware packages rather than the ones from the online repository, e.g.:
```
<repository>\nanoFramework\Tools\nanoff --suppressnanoffversioncheck --listtargets --serialport COM3 --update --fromfwarchive --fwarchivepath <repository>\nanoFramework\Firmware
<repository>\nanoFramework\Tools\nanoff --suppressnanoffversioncheck --platform esp32 --serialport COM3 --update --fromfwarchive --fwarchivepath <repository>\nanoFramework\Firmware
```
The *--suppressnanoffversioncheck* is optional and stops *nanoff* from checking whether a new version of *nanoff* is available. Even if the option is omitted and there is a new version, *nanoff* will not automatically be updated.

### Configure projects

Create a configuration file `<repository>\nanoFramework\nano.devices.json` with the path to the local copy of the Virtual Device and a list of the device types you plan to use. You provide a name for the device type (that is not *Virtual nanoDevice*) and specify the firmware to use, e.g.:
```json
{
    "PathToLocalNanoCLR": "Tools/nanoclr.exe",
    "VirtualDeviceSerialPort": "COM30",
    "FirmwareArchivePath": "Firmware",
    "DeviceTypeTargets": {
        "Primary device": "ESP32_S3_ALL",
        "Alternative": "ESP32_S3_BLE"
    }
}
```

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
    "Platforms": {
        "ESP32"
    }
}
```
The `nano.devices.json` file is used only by the MSBuild task; it is not used by the test platform to determine on which of the connected devices the tests should be executed, and the Visual Studio/VSCode extension allows for the deployment of an application to any connected device.

### Set the default for Virtual Device

Optionally create a `nano.devices.json` file in a folder that contains a Visual Studio solution that contains projects that can be deployed via F5 to a Virtual Device with only:
```json
{
    "GlobalSettingsDirectoryPath": "relative path to <repository>\nanoFramework"
}
```
The Visual Studio extension will read this and use the local copy of the virtual device and serial port as defaults when starting the Virtual Device in the Device Explorer.

### Add NuGet packages to projects

It is possible at any time in the development process to add new NuGet packages for the framework's class libraries to a project, or to upgrade to a newer version. Start by using the latest version and build the project immediately. The MSBuild task will check whether the package version is consistent with the selected versions of the firmware. If that is not the case, install an earlier version of the package and rebuild the project. Repeat until the package version is consistent.

All of the framework's NuGet packages and many packages of the community's packages can be found by searching for the keyword *nanoframework*. To get a list of compatible versions, download nuget.exe at the time the frozen framework version is selected and run:
```
nuget list nanoFramework > <repository>\nanoFramework\NuGetPackageList.txt
```
or
```
nuget list nanoFramework -verbosity detailed > <repository>\nanoFramework\NuGetPackageList.txt
```
The *NuGetPackageList.txt* file lists the version of all framework packages that are compatible with the frozen firmware version.

## Controlled updates for complex projects

The controlled update strategy can also be used for more complex project, e.g., that consists of multiple applications that run on different devices. In the previous section a hierarchical configuration with one global `nano.devices.json` and one file per project was described. In a complex projects the `nano.devices.json` can be used to create a more elaborate hierarchy of configurations.

The MSBuild task starts by reading the `nano.devices.json` file in the project directory. The file can have any of the settings:
```json
{
    "GlobalSettingsDirectoryPath": "relative path to <repository>\nanoFramework",
    "PathToLocalNanoCLR": "Tools/nanoclr.exe",
    "PathToLocalCLRInstanceDirectory": "Firmware/WIN_DLL_nanoCLR-1.12.0.53",
    "VirtualDeviceSerialPort": "COM30",
    "ReservedSerialPorts": ["COM30", "COM31", "COM32", "COM33"],
    "FirmwareArchivePath": "Firmware",
    "DeviceTypeTargets": {
        "Primary device": "ESP32_S3_ALL",
        "Alternative": "ESP32_S3_BLE",
        "Test devices": ["ESP32_S3", "ESP32_S3_ALL", "ESP32_S3_BLE"]
    },
    "DeviceTypes": {
        "Primary device",
        "Virtual nanoDevice"
    },
    "Platforms": {
        "ESP32"
    }
}
```
with:

- `GlobalSettingsDirectoryPath` is the path to the directory containing another `nano.devices.json` file. That file is read first, then the content of this file is used to overwrite the settings per top-level element that is present (*PathToLocalNanoCLR*, *DeviceTypeTargets*, etc.)
- `PathToLocalNanoCLR` is the path to the `nanoclr.exe` file that is used to run the Virtual Device. If it is not present, the global tool is used.
- `PathToLocalCLRInstanceDirectory` is the path to a directory that contains a file `nanoFramework.nanoCLR.dll`. The latter implements an alternative Virtual Device runtime. If this is not present, the runtime embedded in `nanoclr.exe` is used.
- `VirtualDeviceSerialPort` is the serial port to use for a Virtual nanoDevice where applications can be deployed to. The default is "COM30". This setting is used by the Device Explorer of the Visual Studio extension and only has effect if it is present in a `nano.devices.json` in the solution directory or in one of the `nano.devices.json` included by that file.
- `ReservedSerialPorts` is an array of serial ports reserved for use by, e.g., a Virtual nanoDevice. This setting is used by the Device Explorer of the Visual Studio extension; these ports are excluded from the device discovery unless one of the ports is specified as as value for *VirtualDeviceSerialPort*. This allows to have two versions of the framework within a repository, each with its own Virtual nanoDevice running on a dedicated serial port. The setting only has effect if it is present in a `nano.devices.json` in the solution directory or in one of the `nano.devices.json` included by that file.
- `FirmwareArchivePath` is the path to the firmware archive; this is the same path as used in the `--fwarchivepath` argument to *nanoff*.
- `DeviceTypeTargets` is a list of named device types, and per name the name of the runtime/target to use. The name cannot be *Virtual nanoDevice*. The target can be a single name or an array. If this element is present in a global and local `nano.devices.json`, the targets per name are overwritten rather than the *DeviceTypeTargets* list.
- `DeviceTypes` is a list of device types the project is designed to be deployed to. The name *Virtual nanoDevice* refers the the Virtual Device (nanoclr.exe), all other names must have been defined in *DeviceTypeTargets*. 
- `Platforms` is a list of platforms the project is designed to be deployed to. This is shorthand to select all named devices in *DeviceTypeTargets* that match the specified platform.

If the `%USERPROFILE%\.nanoFramework\nano.devices.json` file exists it is always read first, and only the `ReservedSerialPorts` setting is used. This file can be used to exclude other (machine dependent) serial ports that should not be touched by the .NET **nanoFramework** tools.