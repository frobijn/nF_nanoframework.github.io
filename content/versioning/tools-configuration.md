# Tools and configuration

The .NET **nanoFramework** has out-of-the-box support for the [auto-update](versioning-strategies.md#auto-update) versioning strategy. For the other supported strategies some extra configuration files and tools have to be added.

The configuration is also used by other tools that require information about the devices relevant for a project. These tools include the [Visual Studio extension](../getting-started-guides/getting-started-managed.md) and the latest version of the [test framework](../unit-test/framework-v3). Custom tools can use the configuration by using the [nanoFramework.Tooling.Devices](https://github.com/nanoframework/nf-tools) library.

The configuration consists of several components, depending on the adopted versioning strategy.

![Configurations for versioning strategies](../../images/versioning-devices-configuration.png)

- A [firmware archive](#firmware-archive) directory that contains the firmware packages for the device types your software is intended to be deployed to.
- A [local tools](#local-tools) directory; only required for the controlled update strategy.
- A file (e.g., *NuGetPackages.txt*) that lists the [NuGet package versions](#nuget-package-list) to be used in .NET **nanoFramework** projects.
- A description of the [configuration and devices](#configuration-and-devices) as stored in `nano.devices.json` files.

The first three components are created by running commands. These can be combined in an [update script](#update-script) that, in case of the daily update strategy, can be run every day.

A [NuGet package](#consistency-verification-task) should be added to each .NET **nanoFramework** project for which a consistency check is required. The package installs a MSBuild task that verifies the consistency of the NuGet packages used by the project with the packages in the NuGet package list and the firmware for the devices the projects is intended to be deployed to.

## Local tools 

You only have to create local copies of tools if you adopt the controlled update strategy.

In that case use a local copy of the [nanoff](https://github.com/nanoframework/nanoFirmwareFlasher) tool to deploy firmware, applications and/or files to a device. The tool has some business logic about the firmware to select and about partitioning of the device's flash that may be different in later versions. Install a local copies of the .NET **nanoFramework** tool:

```
dotnet tool install nanoff --tool-path <repository>\.nanoFramework\Tools
```

## Firmware archive

Tools like the [consistency verification task](#consistency-verification-task) require that the firmware packages are available locally. If you adopt the controlled update strategy, you'll likely store the firmware packages in a subdirectory of the product's repository. If you adopt the daily update strategy, a subdirectory of your user profile directory is a good option, e.g., `%USERPROFILE%\.nanoFramework\Firmware`.

Collect the firmware packages you need for your devices or (in case of a product) the devices the software is intended to be deployed to. You can use the global *nanoff* tool for that. E.g.:

```
nanoff --target ESP32_S3_ALL --updatearchive --archivepath <archive-directory>
```

If you don't know what firmware to use but have a new device connected to the PC, you may find the best matching firmware via:
```
nanoff --platform ESP32 --identifyfirmware --serialport COM9
```
This works for selected platforms only.
If you are not sure which devices you will be using, collect all packages for a platform.
```
nanoff --platform ESP32 --updatearchive --archivepath <archive-directory>
```

If you adopt the controlled update strategy and use the Virtual nanoDevice for testing, you have to archive the Virtual nanoDevice runtime (*WIN_DLL_nanoCLR*) as well:
```
nanoff --target WIN_DLL_nanoCLR --updatearchive --archivepath <archive-directory>
```

From now on (for all strategies), use the extra *archive*-options to use the downloaded firmware instead of the online repository, e.g.:
```
nanoff --listtargets --serialport COM3 --update --fromarchive --archivepath <archive-directory>
nanoff --platform esp32 --serialport COM3 --update --fromarchive --archivepath <archive-directory>
```

If you adopt the controlled update strategy, use the local copy of *nanoff* instead of the global tool.

## NuGet package list

The [consistency verification task](#consistency-verification-task) checks whether a project uses the specified NuGet package versions for white-listed packages. The list of packages is a text file with the identification of the package (e.g., `nanoFramework.CoreLibrary`) at the start of a line, followed by whitespace (including end of line) and the required version number of the package.

If you adopt the controlled update strategy, you'll likely store the list of NuGet packages in a subdirectory of the product's repository. If you adopt the daily update strategy, a subdirectory of your user profile directory is a good option, e.g., `%USERPROFILE%\.nanoFramework\NuGetPackages.txt`. This description uses *NuGetPackages.txt* as file name, but you can use any file name you like.

All of the framework's NuGet packages and many packages of the community's packages can be found by searching for the keyword *nanoframework*. To get a list of the versions that are current, download nuget.exe and run:
```
nuget list nanoFramework > <directory>\NuGetPackages.txt
```
or
```
nuget list nanoFramework -verbosity detailed > <directory>\NuGetPackages.txt
```
You'll notice that there is a warning at the top of *NuGetPackages.txt* that you should use the *search* option rather than *list*. Don't do that, the *search* option only finds a subset of all packages (at the time of writing).

## Update script

If you adopt the daily update strategy, create a script that updates the global tools, selected firmware packages and NuGet package list. E.g.:

```
@echo off

dotnet tool update -g nanoff
dotnet tool update -g nanoclr

nanoff --suppressnanoffversioncheck --updatearchive --removeoldversions --target ESP32_S3_ALL --archivepath "%USERPROFILE%\.nanoFramework\Firmware"
nanoff --suppressnanoffversioncheck --updatearchive --removeoldversions --target ESP32_C6_THREAD --archivepath "%USERPROFILE%\.nanoFramework\Firmware"
rem... more targets ...

"%USERPROFILE%\.nanoFramework\nuget.exe" list nanoFramework -verbosity detailed > "%USERPROFILE%\.nanoFramework\NuGetPackages.txt"
```

Use the Task Scheduler in Windows (or a similar program) to run this script every day. Avoid running the script around midnight UTC, as the firmware and NuGet packages may be in the process of being updated by the .NET **nanoFramework**'s daily build pipelines.


## Configuration and devices

The (type of) nanoDevices a project's software can be deployed to is specified in a [`nano.devices.json`](nano-devices-json.md) file. Relevant for the [Consistency verification task](#consistency-verification-task) are:

- `NuGetPackageList` is required and specifies the location of the NuGet packages whitelist.
- `NanoCLRPath` is optional ans specifies the location of a local version of `nanoclr.exe`.
- `FirmwareArchivePath` is required and specifies the location of the firmware archive.
- One or more of:
    - `DeviceTypeTargets` and `DeviceTypes` select the nanoDevices by the name of the firmware/target.
    - `Platforms` select the nanoDevices by the name of the firmware/target.
    - `DeviceSelection` and `Devices` select nanoDevices by system serial number or module serial number.

To prevent duplication of settings in many project configuration files, a `nano.devices.json` can import other `nano.devices.json` files. Settings like *NuGetPackageList*, *FirmwareArchivePath* and *DeviceTypes* can be stored in a global configuration file that is imported by the project's configuration. The figure at the top of the page illustrates how such an hierarchy of configuration files could look like. See the [`nano.devices.json`](nano-devices-json.md) for details.

## Consistency verification task

The consistency verification task checks that white-listed NuGet packages used by a .NET **nanoFramework** project have the correct version. It also verifies that all NuGet packages are consistent with the firmware of the device types the project is intended to be deployed to.

Install the task by adding the `nanoFramework.Versioning` NuGet package to the project. The task requires that the project has a `nano.devices.json` configuration with valid *NuGetPackageList* and *DeviceTypes* entries.
