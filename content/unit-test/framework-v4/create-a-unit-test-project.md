# Create a unit test project

## Create a testable project

Before even considering to create a unit test project, you should consider whether the project(s) to create the unit tests for satisfies the criteria of a testable project:

- The project is a class library.

	It is not possible to create unit tests for .NET nanoFramework applications. Instead, create a class library first, move all testable code to the library and create a unit test project for that library.

- The project is suited for the device you want to test on.

	It is possible to execute and debug unit tests on both the Virtual nanoDevice and on real hardware nanoDevices. But like all .NET **nanoFramework** code, it is not possible to execute an assembly on a device if the firmware/target installed on the device does not support all required features. If the assembly being tested depends on class libraries with hardware-specific and/or platform specific features, the tests can only be executed on that hardware or devices from that platform.

Do not write (unit) tests to verify whether software can be deployed to a particular type of device. The .NET **nanoFramework** has [other tooling](../../versioning) that perform the check as part of the build process.

### Example: appliance with complex business logic

Imagine you are working on the software for an appliance that has several sensors. The software reads data from the sensors and periodically sends the data to a cloud server for further processing. It also contains (complex) business logic to analyze the data read from the sensors and turns on an alarm in case of dangerous levels are measured.

To make the software testable you would split the software in at least three projects:

- A class library that contains the business logic for data processing and analysis. It defines interfaces to access the sensors; mocks or fakes are provided for the interfaces in the tests. The class library has no hardware dependency and the tests for this library can be executed and debugged on the (fast!) Virtual nanoDevice.

- A class library that implements the interfaces and contains code to access the sensors, send data to a cloud server, and other features that require real hardware. The tests for this library will be executed on real hardware nanoDevices.

- An application that initialises the device it is running on and delegates the execution to code in the class libraries. The application cannot be tested and has as few lines of code as possible.

## Create a new unit test project

The easiest way to create a new unit test project is to use the project templates provided by the Visual Studio extension. 

- Select *File | New | Project* in Visual Studio

- Filter the project templates by platform: *nanoFramework*

- Select *Unit Test Project (.NET nanoFramework)*

After the project is created, add the .NET nanoFramework class library that you want to test as a reference.

If the project the tests are created for is using a [`nano.devices.json` configuration](../../versioning), make sure to create a `nano.devices.json` configuration for the test project as well.

## Create as class library project

A unit test project is just a .NET nanoFramework class library that references the `nanoFramework.TestFramework.TestAdapter` NuGet package. If that suites you better, you can start by creating a class library project and then add the NuGet package.
