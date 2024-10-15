# Versioning and consistency checks

The .NET **nanoFramework** consists of multiple components that are required to create, build, test and deploy an application. At first sight it may seem that the various components are updated independently of each other, and that you as a developer have the freedom to mix and match component versions. That is not true: if you are not careful, you'll end up with an application that cannot be deployed to the device of your choice. If you want to know more about details the components, tools and their interdependencies: that is explained in the [architecture](../architecture/deployment.md) section.

Fortunately, the .NET **nanoFramework** offers several strategies and tools to help you select the right versions of the components without having to fully understand how the framework is organized:

- [Versioning strategy](versioning-strategies.md): when to update framework tools and components. Can you always use the latest versions of the framework, or do you need time to prepare for an update? 

- [Consistency checks](consistency-checks.md): when to verify that an application can be deployed to a type of device? Is your development time short enough to do the check when the application is deployed to a device? Or do you want to check often and early in the development process?

If you choose to always work with the latest versions and do the consistency checks when the application is deployed to a device, you don't have to any extra work. This is supported out of the box. Otherwise you need to add extra files to your projects:

- [Tools and configuration](tools-configuration.md) to support controlled updates and/or early consistency checks.
