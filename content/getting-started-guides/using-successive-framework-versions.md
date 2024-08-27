# Using successive .NET nanoFramework versions (C#)

Writing, testing and deploying a managed code application requires three groups of components provided by the .NET **nanoFramework** and by community members:

- The application uses managed class libraries that implement the .NET **nanoFramework**. The class libraries are provided as NuGet-packages.
- The .NET **nanoFramework** provides tools to test an application on a (Windows) machine and deploy the application and firmware to an embedded device. The tools, [nanoFirmwareFlasher](getting-started-managed#uploading-the-firmware-to-the-board-using-nanofirmwareflasher) and [.NET nanoFramework Virtual Device](virtual-device), are distributed as `dotnet` tool.
- An embedded device requires firmware to run the application. The firmware is a native implementation of the .NET **nanoFramework** for that particular device. The firmware is obtained by the nanoFirmwareFlasher from an online repository.

The components have mutual dependencies. To use a new feature in a class library a new version of the firmware is required that implements that feature for a device. A new firmware version may be released that fixes bugs in a previous version. A new type of device may require new firmware and a new version of the deployment tool. As a developer you have to be careful to use only component versions that are compatible with each other. The development tooling will help you to some extent, but tracking interdependencies of components from different groups is not straightforward. 

As a developer who just wants to use the .NET **nanoFramework** to create an exiting application, keeping track of all these versions may seem a bit daunting. If you pick one of the supported strategies, you can avoid all that hard work and be sure your application is using only components that are made for each other:

1. Always use the latest and greatest versions

2. Keep using the same version until there is a reason to upgrade to the latest version.

## Strategy 1: always use the latest and greatest versions
The easiest strategy is to always use the latest version of all components. You'll always have access to all features in the class library, the latest implementations for a particular device and the widest range of devices to deploy your application to.

When you [get started](getting-started-managed) to use the .NET **nanoFramework**, you install the tools as global dotnet tools. You only have to install the nanoFirmwareFlasher:

```
	dotnet tool -g install nanoff
```

In Visual Studio, the project created for an application, class library or unit tests already depends on the core .NET **nanoFramework** libraries that are automatically retrieved by the NuGet package manager. If you need extra features, you install the latest version of the class library that implements the feature. If new versions of the packages become available, you upgrade to the latest version.

If the application or unit tests are ready to be run or tested on a new embedded device, use the [nanoff](https://github.com/nanoframework/nanoFirmwareFlasher) tool to install the latest version of the firmware for that device, e.g.:
```
	nanoff --platform esp32 --serialport COM3 
```
The application or unit tests can be deployed to the device using the Visual Studio extension.

If the deployment fails with a message about incompatible versions, you've most likely missed a new version of the firmware of class libraries. Go to the NuGet package management for the solution in Visual Studio and check whether there are updates for the packages (and install any updates). If that does not solve the problem, update the firmware, e.g.:
```
	nanoff --platform esp32 --serialport COM3 --update
```

If you want the first to know when new components are available, join us on [Discord](../contact-2). Or remember to check for new versions regularly, e.g., some time before you plan to distribute a new release of your application. 

## Strategy 2: keep using the same version until there is a reason to upgrade to the latest version
The first strategy is not suited if an update to new versions of components is not easy. There may be several reasons for that: the devices may not be easily accessible, the application is maintained but not actively developed, or an upgrade affects multiple teams and requires formal coordination. Then the second strategy can be applied: keep using the same version until there is a reason to upgrade to the latest version. Typical reasons to upgrade are that the application has to be deployed to a new type of device, a new feature is added to the application that requires a .NET **nanoFramework** feature that hitherto has not been used, or the application suffers from a bug in the firmware for which a new version is available. New versions of components that have no effect on the application's performance are usually ignored.

The first step is to decide how to keep access to the single version of the .NET **nanoFramework** components:

- The class libraries are obtained from NuGet.org. The general expectation is that these versions will be available forever. If you don't want to depend on that, make a backup of the NuGet packages your application is using.

- The nanoFirmwareFlasher and .NET **nanoFramework** Virtual Device are also distributed as NuGet package. Alternatively store the tools as part of your application's code repository.

- The nanoFirmwareFlasher can back up the firmware components in a designated directory and use it as source for the installation of firmware on devices. Decide how to back up the components in this directory.

If you start work on the application or decide to upgrade to the latest version, make sure you have everything you need to keep working with this version:

- Install the nanoFirmwareFlasher and .NET **nanoFramework** Virtual Device in a directory that is specific for your project, e.g.:
	```
	dotnet tool install nanoff --tool-path c:\MySolution\nanoFramework
	dotnet tool install nanoclr --tool-path c:\MySolution\nanoFramework
	```
	Backup the tools and/or the NuGet packages and use the backup for installation of the tools on other machines.

- Create a list of the versions of all NuGet packages of the .NET **nanoFramework** class libraries and community additions. The easiest way is to download NuGet.exe and run:
	```
	nuget list nanoFramework > c:\MySolution\nanoFramework\NuGetPackageList.txt
	```
	The resulting file contains a list of all packages with *nanoFramework* in the description, and the current version of the package. You can include a short description by running:
	```
	nuget list nanoFramework -verbosity detailed > c:\MySolution\nanoFramework\NuGetPackageList.txt
	```

- If you already have code that uses the .NET **nanoFramework**, go to the NuGet package management for each solution in Visual Studio and upgrade to the latest version of the packages that implement or are based on the .NET **nanoFramework**.

- Retrieve the firmware for all devices that you are using or plan to use. For each device or platform run nanoff with the --fwarchivepath and --updatefwarchive options, e.g.:
	```
	c:\MySolution\nanoFramework\nanoff --target ESP32_S3_ALL --updatefwarchive --fwarchivepath c:\MySolution\nanoFramework\Firmware 
	```
	or, if the device is connected to the PC:
	```
	c:\MySolution\nanoFramework\nanoff --nanodevice --serialport COM3 --updatefwarchive --fwarchivepath c:\MySolution\nanoFramework\Firmware
	```	The device itself is not updated by this command.

	Make sure the archive directory passed via the --fwarchivepath option is empty before retrieving the new firmware versions. Archive its contents after all firmware is retrieved.

Now create projects for your application and optionally class libraries and unit tests and code the application. Make sure that:

- Your projects should only use versions of the NuGet packages that are in the previously created list of packages (e.g., c:\MySolution\nanoFramework\NuGetPackageList.txt).
- For every (new) .NET **nanoFramework** unit test project update the `nano.runsettings` file in the root of the project directory and set:
	```
	<PathToLocalCLRInstance>c:\MySolution\nanoFramework\nanoclr.exe</PathToLocalCLRInstance>
	```
	Preferably use a path relative to the directory that contains the `nano.settings` file, e.g.:
	```
	<PathToLocalCLRInstance>..\nanoFramework\nanoclr.exe</PathToLocalCLRInstance>
	```
	This can be omitted if the unit tests from the project will only ever be run on a real device.

To deploy firmware to a device, use the same options as for strategy 1 but add the --fwarchivepath and --fromfwarchive options, e.g.: 
```
c:\MySolution\nanoFramework\nanoff --suppressnanoffversioncheck --platform esp32 --serialport COM3 --update --fromfwarchive --fwarchivepath c:\MySolution\nanoFramework\Firmware
```
An application can be deployed to a device using the Visual Studio extension, provided the firmware on the device has been updated using te previous command. An application can also be deployed using the project-specific version of the nanoFirmwareFlasher as long as the --fwarchivepath and --fromfwarchive options are added.

You can use the nanoFirmwareFlasher in the same way with the same arguments as normal, provided you use the local nanoff application and add the --fromfwarchive and --fwarchivepath options. E.g.:
```
c:\MySolution\nanoFramework\nanoff --suppressnanoffversioncheck --listtargets --fromfwarchive --fwarchivepath c:\MySolution\nanoFramework\Firmware
```
lists the non-preview firmware versions in the archive directory.


## What about the Visual Studio extension?
The Visual Studio extension does not depend on the .NET **nanoFramework** class libraries or its implementation (firmware or .NET **nanoFramework** Virtual Device). Regardless of the chosen strategy you can decide to update the extension as soon as a new version is available, not at all or something inbetween.