# Versioning strategy

## Affected components

To develop, test and deploy an application to a device, you'll need several components and tools. For some tools it is not relevant which version you are using:

- [Visual Studio extension](../getting-started-guides/getting-started-managed.md);
- [Test framework](../unit-test);
- The Virtual Device host [nanoclr.exe](../getting-started-guides/virtual-device.md); the version of the included firmware does matter.

These tools can be updated at any time and keep working with the installed versions of other .NET **nanoFramework** components. The Visual Studio extension is by default auto-updated by Visual Studio. The test framework is a NuGet package that can be updated whenever a new package is available. The Virtual Device is a .NET tool that can be updated manually, and is also updated if it is used by the Visual Studio extension and test framework.

The other tools and components cannot bee freely updated, as there are dependencies between the various versions of the components:

- NuGet packages with the .NET class libraries.
- Firmware packages for the various device types/targets.
- Firmware package for the Virtual Device.
- The [nanoff](https://github.com/nanoframework/nanoFirmwareFlasher) tool to deploy applications and firmware to a device.

## Supported strategies
How can you make sure that your application uses matching versions of the NuGet and firmware packages and of *nanoff*? 

The CI/CD pipeline of the .NET **nanoFramework** ensures that all packages and tools are consistent with each other at any moment in time. All packages and tools are updated daily (around 0:00 UTC) with the latest enhancements and bug fixes, and if packages depend on each other, the references are updated. In principle, that is. Some tools and packages hardly ever change, there are many days without updates. But if there are enhancements and bug fixes, they will be made available as quickly as possible.

If two packages have been updated, then both old versions of the packages are consistent and both new versions. Although the new version of one package may be consistent with the old version of the other package, there is no way of knowing that for sure. Neither the CI/CD pipelines nor the core team and contributors will test such a combination.

The .NET **nanoFramework** therefore supports two versioning strategies and a mix of both:

- [Auto-update](#auto-update): always use the latest version. If one of the framework tools signals a version discrepancy, simply update to the latest version. This is the easiest way to use the framework.
- [Controlled update](#controlled-update): select a single version of all package and tools to develop and test your application. Plan to update regularly to the latest version, but update the packages and tools at a time that suites you. Use this strategy if you require more control over the update process.
- [Daily update](#daily-update): a mix of both. Configure your projects to verify the consistency of the various components, like the *controlled update* strategy. Update the selected single version of the components every day, like the auto-update strategy. Use this strategy if you want to have consistency checks early in the development process.

By default the .NET **nanoFramework** is configured for the auto-update strategy. You have to [add support](tools-configuration.md) to your projects, solution and/or repository if you want to select one of the other strategies.

## Auto-update

If you choose to always use the latest versions of the framework tools and components, this is what you should do.

If you deploy firmware to a device with *nanoff*, the latest version of the firmware will be used. If there is an update of *nanoff* itself, the tool will tell you if you run it.

Updates of the NuGet packages are available via the package manager in Visual Studio. If there are new versions of the packages, always update to the latest version. You have to check for updates by hand, via the NuGet package manager. 

If you deploy your application (via F5 in Visual Studio) to a device, a version check is done to verify that the NuGet packages used by the application are consistent with the firmware installed on the device. If there is a mismatch, an error message tells you whether the NuGet package or the firmware is more recent. Just update the NuGet package or the firmware to the latest version, and you're good to go.

At the moment of writing, the version check is not executed if you deploy an application via *nanoff* to a device. If you want to deploy an application with *nanoff* to a new device, always include an update of the firmware as well. Make sure you have tested before (via F5 in Visual Studio) whether the latest firmware and your applications are consistent.

## Controlled update

Especially in commercial projects the auto-update strategy may not be feasible. An update of one of the packages or firmware may require planning and coordination with other teams. You and all your colleagues must use the same version of the framework's packages and tools. You should still plan to update to the latest version of the framework regularly, if only to profit from bug fixes. But the actual update to a new version (if there is one) is planned and executed when there is time to assert that your application and device is still working correctly after the update.

Instead of using the latest version, you are using a version that is frozen in time. At the start of a project, you make a local copy of the firmware packages for your devices and of the *nanoff* tool and Virtual Device runtime. These are stored in a project-specific location. You only use the NuGet packages that are current at that time, or at least NuGet packages that match the frozen firmware. If you deploy firmware to a (new) device, you'll use the local copy of *nanoff* and of the firmware. In unit testing and debugging you use the local copy of the Virtual Device runtime.

While you are developing, it may become necessary to use additional NuGet packages. Or you may encounter an issue with one of the NuGet packages that can be resolved by a bug fix of the class library only. (The community welcomes any PR with a resolution of an issue!) If your project allows to start using new (versions of) NuGet packages, you want to be sure from the start that the packages are consistent with the frozen version of the firmware. Add some [extra files](tools-configuration.md) to each project, and when the project is built the consistency of NuGet packages and the selected firmware versions is verified.

If you are ready to update to a new version, freeze the then-current version of the NuGet and firmware packages and tools. Rebuilt your projects and update NuGet packages if necessary. Deploy the new firmware to test devices, and optionally re-run your test projects. If all tests succeed, keep the version and proceed to develop/test with that version.

## Daily update

The daily update strategy combines the advantage of working with the latest versions from the auto-update strategy with the consistency verification from the controlled update strategy.

You create a script or program that updates the global *nanoff* and *nanoclr* tools. The *nanoff* tool contains the latest version of the Virtual Device runtime. You also make a local copy of the firmware packages for your devices, but use a global archive directory that is shared by all projects.

Use the Windows Task Scheduler or a similar program to run the script once a day. Avoid running the script around midnight UTC as the .NET **nanoFramework** pipelines may be active at that time.

You add some [extra files](tools-configuration.md) to each project, and when the project is built the consistency of NuGet packages and the selected firmware versions is verified.

If you have not modified an application recently, re-build the application before deploying it to a device to make sure your application uses the latest NuGet packages. If you deploy the application with *nanoff* to a device, be sure to include an update of the firmware.

